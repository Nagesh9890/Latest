from pyspark.sql import SparkSession
from pyspark.sql.functions import col, lower
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.linalg import Vectors
from pyspark.ml import Pipeline
import pickle

# Initialize a Spark session
spark = SparkSession.builder.appName("YourAppName").getOrCreate()

# Load the saved models and vectorizers
clf_cat1 = pickle.load(open("clf_cat1.pkl", "rb"))
clf_cat2 = pickle.load(open("clf_cat2.pkl", "rb"))
tfidf_payer_name = pickle.load(open("tfidf_payer_name.pkl", "rb"))
tfidf_payee_name = pickle.load(open("tfidf_payee_name.pkl", "rb"))
tfidf_payee_account_type = pickle.load(open("tfidf_payee_account_type.pkl", "rb"))
tfidf_payer_account_type = pickle.load(open("tfidf_payer_account_type.pkl", "rb"))
tfidf_payer_vpa = pickle.load(open("tfidf_payer_vpa.pkl", "rb"))
tfidf_payee_vpa = pickle.load(open("tfidf_payee_vpa.pkl", "rb"))

# Define a function to preprocess and predict the categories
def preprocess_and_predict(payer_name, payee_name, payee_account_type,
                            payer_account_type, payer_vpa, payee_vpa):
    # Check for None values and convert to lowercase
    payer_name = payer_name.lower() if payer_name else ""
    payee_name = payee_name.lower() if payee_name else ""
    payee_account_type = payee_account_type.lower() if payee_account_type else ""
    payer_account_type = payer_account_type.lower() if payer_account_type else ""
    payer_vpa = payer_vpa.lower() if payer_vpa else ""
    payee_vpa = payee_vpa.lower() if payee_vpa else ""

    # Transform input data using the TFIDF vectorizers
    payer_name_vec = tfidf_payer_name.transform([payer_name])
    payee_name_vec = tfidf_payee_name.transform([payee_name])
    payee_account_type_vec = tfidf_payee_account_type.transform([payee_account_type])
    payer_account_type_vec = tfidf_payer_account_type.transform([payer_account_type])
    payer_vpa_vec = tfidf_payer_vpa.transform([payer_vpa])
    payee_vpa_vec = tfidf_payee_vpa.transform([payee_vpa])

    # Combine all features into a single vector
    feature_vector = Vectors.dense(
        payer_name_vec.toArray().tolist() +
        payee_name_vec.toArray().tolist() +
        payee_account_type_vec.toArray().tolist() +
        payer_account_type_vec.toArray().tolist() +
        payer_vpa_vec.toArray().tolist() +
        payee_vpa_vec.toArray().tolist()
    )

    # Predict
    prediction_cat1 = clf_cat1.predict([feature_vector])
    prediction_cat2 = clf_cat2.predict([feature_vector])

    return [str(prediction_cat1[0]), str(prediction_cat2[0])]

# Make sure your DataFrame has columns: payer_name, payee_name, payee_account_type, payer_account_type, payer_vpa, payee_vpa
# Assuming df2 is your input DataFrame
for col_name in ["payer_name", "payee_name", "payee_account_type", "payer_account_type", "payer_vpa", "payee_vpa"]:
    df2 = df2.withColumn(col_name, lower(col(col_name)))

# Define the prediction UDF
predict_udf = udf(preprocess_and_predict, ArrayType(StringType()))

result_df = df2.withColumn("predictions", predict_udf("payer_name", "payee_name", "payee_account_type", "payer_account_type", "payer_vpa", "payee_vpa"))

# Add the "category_level1" and "category_level2" columns
result_df = result_df.withColumn("category_level1", result_df["predictions"].getItem(0))
result_df = result_df.withColumn("category_level2", result_df["predictions"].getItem(1))

# Dropping the combined predictions column
result_df = result_df.drop("predictions")

# Show the resulting DataFrame
result_df.show()
