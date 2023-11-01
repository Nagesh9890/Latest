#automated Model Training
#Importing Libraries
import numpy as np
import pandas as pd
import pickle
from sklearn.cross_validation import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import confusion_matrix
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics import accuracy_score

df = pd.read_excel("PhonePe_Sherloc_Categories.xlsx")  # Replace with the path to your dataset
df.columns
df2 = df[['payer_name','payer_vpa','payee_account_type','payee_name','payee_vpa','payer_account_type', 'Category1','Category2']]

def custom_tokenizer(text):
    # split the text and value using regular expression
    import re
    pattern = re.compile(r'[a-zA-Z]+\d+')
    text_and_value = pattern.findall(text)
    return text_and_value
# Apply TF-Vectorization on data
tfidf_payer_name = TfidfVectorizer()
tfidf_matrix_payer_name = tfidf_payer_name.fit_transform(df2['payer_name'].astype(str))

tfidf_payee_name = TfidfVectorizer()
tfidf_matrix_payee_name = tfidf_payee_name.fit_transform(df2['payee_name'].astype(str))

tfidf_payee_account_type = TfidfVectorizer()
tfidf_matrix_payee_account_type = tfidf_payee_account_type.fit_transform(df2['payee_account_type'].astype(str))

tfidf_payer_account_type = TfidfVectorizer()
tfidf_matrix_payer_account_type = tfidf_payer_account_type.fit_transform(df2['payer_account_type'].astype(str))

tfidf_payer_vpa = TfidfVectorizer(tokenizer=custom_tokenizer)
df2['payer_vpa'] = df2['payer_vpa'].astype(str)
tfidf_matrix_payer_vpa = tfidf_payer_vpa.fit_transform(df2['payer_vpa'])

tfidf_payee_vpa = TfidfVectorizer(tokenizer=custom_tokenizer)
df2['payee_vpa'] = df2['payee_vpa'].astype(str)
tfidf_matrix_payee_vpa = tfidf_payee_vpa.fit_transform(df2['payee_vpa'])


tfidf_matrix = pd.concat([pd.DataFrame(tfidf_matrix_payer_name.toarray()),
                          pd.DataFrame(tfidf_matrix_payee_name.toarray()),
                          pd.DataFrame(tfidf_matrix_payee_account_type.toarray()),
                          pd.DataFrame(tfidf_matrix_payer_account_type.toarray()),
                          pd.DataFrame(tfidf_matrix_payer_vpa.toarray()),
                          pd.DataFrame(tfidf_matrix_payee_vpa.toarray())], axis=1)
X_train, X_test, y_cat1_train, y_cat1_test, y_cat2_train, y_cat2_test = train_test_split(tfidf_matrix, df2['Category1'], df2['Category2'], test_size=0.2, random_state=42)
clf_cat1 = RandomForestClassifier()

clf_cat1.fit(X_train, y_cat1_train)

clf_cat2 = RandomForestClassifier()
clf_cat2.fit(X_train, y_cat2_train) # Make predictions for each target variable

predictions_cat1 = clf_cat1.predict(X_test)
predictions_cat2 = clf_cat2.predict(X_test)

accuracy_cat1 = accuracy_score(y_cat1_test, predictions_cat1)
print "Accuracy for Category Level 1: %.2f" % accuracy_cat1 


# Calculate accuracy for Category Level 2 predictions
accuracy_cat2 = accuracy_score(y_cat2_test, predictions_cat2)
print "Accuracy for Category Level 2: %.2f" % accuracy_cat2

df2.head(2)

# After training the Random Forest classifiers:
with open("clf_cat1.pkl", "wb") as f:
    pickle.dump(clf_cat1, f)

with open("clf_cat2.pkl", "wb") as f:
    pickle.dump(clf_cat2, f)

# After fitting the Tfidf vectorizers:
with open("tfidf_payer_name.pkl", "wb") as f:
    pickle.dump(tfidf_payer_name, f)

with open("tfidf_payee_name.pkl", "wb") as f:
    pickle.dump(tfidf_payee_name, f)

with open("tfidf_payee_account_type.pkl", "wb") as f:
    pickle.dump(tfidf_payee_account_type, f)

with open("tfidf_payer_account_type.pkl", "wb") as f:
    pickle.dump(tfidf_payer_account_type, f)

with open("tfidf_payer_vpa.pkl", "wb") as f:
    pickle.dump(tfidf_payer_vpa, f)

with open("tfidf_payee_vpa.pkl", "wb") as f:
    pickle.dump(tfidf_payee_vpa, f)



















# Define the custom tokenizer
def custom_tokenizer(text):
    pattern = re.compile(r'[a-zA-Z]+\d+')
    return pattern.findall(text)

# Load the saved models and vectorizers
clf_cat1 = pickle.load(open("clf_cat1.pkl", "rb"))
clf_cat2 = pickle.load(open("clf_cat2.pkl", "rb"))
tfidf_payer_name = pickle.load(open("tfidf_payer_name.pkl", "rb"))
tfidf_payee_name = pickle.load(open("tfidf_payee_name.pkl", "rb"))
tfidf_payee_account_type = pickle.load(open("tfidf_payee_account_type.pkl", "rb"))
tfidf_payer_account_type = pickle.load(open("tfidf_payer_account_type.pkl", "rb"))
tfidf_payer_vpa = pickle.load(open("tfidf_payer_vpa.pkl", "rb"))
tfidf_payee_vpa = pickle.load(open("tfidf_payee_vpa.pkl", "rb"))

# Define the prediction function with handling for None values
def predict_categories(payer_name, payee_name, payee_account_type,
                       payer_account_type, payer_vpa, payee_vpa):
    
    # Handle potential None values
    payer_name = '' if payer_name is None else payer_name
    payee_name = '' if payee_name is None else payee_name
    payee_account_type = '' if payee_account_type is None else payee_account_type
    payer_account_type = '' if payer_account_type is None else payer_account_type
    payer_vpa = '' if payer_vpa is None else payer_vpa
    payee_vpa = '' if payee_vpa is None else payee_vpa
    
    # Transform input data using the TFIDF vectorizers
    payer_name_vec = tfidf_payer_name.transform([payer_name])
    payee_name_vec = tfidf_payee_name.transform([payee_name])
    payee_account_type_vec = tfidf_payee_account_type.transform([payee_account_type])
    payer_account_type_vec = tfidf_payer_account_type.transform([payer_account_type])
    payer_vpa_vec = tfidf_payer_vpa.transform([payer_vpa])
    payee_vpa_vec = tfidf_payee_vpa.transform([payee_vpa])

    tfidf_matrix = pd.concat([pd.DataFrame(payer_name_vec.toarray()),
                              pd.DataFrame(payee_name_vec.toarray()),
                              pd.DataFrame(payee_account_type_vec.toarray()),
                              pd.DataFrame(payer_account_type_vec.toarray()),
                              pd.DataFrame(payer_vpa_vec.toarray()),
                              pd.DataFrame(payee_vpa_vec.toarray())], axis=1)

    # Predict
    prediction_cat1 = clf_cat1.predict(tfidf_matrix)
    prediction_cat2 = clf_cat2.predict(tfidf_matrix)

    return [prediction_cat1[0], prediction_cat2[0]]

# Register the prediction function as a UDF
predict_udf = udf(predict_categories, ArrayType(StringType()))

# Assuming df2 is already defined and contains the necessary columns
result_df = df2.withColumn("predictions", predict_udf("payer_name", "payee_name", "payee_account_type", "payer_account_type", "payer_vpa", "payee_vpa"))

# Splitting predictions into individual columns
result_df = result_df.withColumn("category_level1", result_df["predictions"].getItem(0))
result_df = result_df.withColumn("category_level2", result_df["predictions"].getItem(1))

# Dropping the combined predictions column
result_df = result_df.drop("predictions")

# Show the resulting DataFrame
result_df.show(10)
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  --------------------------------------
  File "<ipython-input-12-c46e6c75c523>", line 20
    .otherwise(F.expr(f"CASE WHEN sum_{category} BETWEEN 5000000 * (10 - id) AND 5000000 * (9 - id) THEN CAST(id AS STRING) ELSE NULL END"))
                                                                                                                                         ^
SyntaxError: invalid syntax

from pyspark.sql import functions as F
from pyspark.sql import Window

def compute_aggregates_optimized(df, category_col, amount_col):
    # Pivot and aggregate
    agg_df = df.groupBy("payer_account_number", "payer_account_type").pivot(category_col).agg(
        F.count(amount_col).alias("count"),
        F.sum(amount_col).alias("sum")
    )
    
    # List of categories
    categories = df.select(category_col).distinct().rdd.flatMap(lambda x: x).collect()
    
    # Rename columns, and add value type column
    for category in categories:
        agg_df = agg_df.withColumnRenamed(category + "_count", "count_" + category) \
                      .withColumnRenamed(category + "_sum", "sum_" + category) \
                      .withColumn("type_" + category, 
                                  F.when((F.col("count_" + category) == 0) | (F.col("sum_" + category).isNull()) | (F.col("sum_" + category) == 0), "No transactions")
                                  .when(F.col("payer_account_type") == "SAVINGS", 
                                        F.when(F.col("sum_" + category) < 5000000, "10")
                                        .otherwise(F.expr(f"CASE WHEN sum_{category} BETWEEN 5000000 * (10 - id) AND 5000000 * (9 - id) THEN CAST(id AS STRING) ELSE NULL END"))
                                  .when(F.col("payer_account_type") == "CURRENT", 
                                        F.when(F.col("sum_" + category) < 15000000, "10")
                                        .otherwise(F.expr(f"CASE WHEN sum_{category} BETWEEN 15000000 * (10 - id) AND 15000000 * (9 - id) THEN CAST(id AS STRING) ELSE NULL END"))
                                  .otherwise("Unknown Type"))
    
    return agg_df.cache()  # Cache this DataFrame

# Compute aggregates for category_level1 and category_level2 
agg_df1 = compute_aggregates_optimized(result_df, "category_level1", "payer_amount")
agg_df2 = compute_aggregates_optimized(result_df, "category_level2", "payer_amount")

# Join the two aggregated DataFrames 
final_agg_df = agg_df1.join(F.broadcast(agg_df2), ["payer_account_number", "payer_account_type"], "outer").fillna(0)

# Calculate the total transaction sum and appearance for each account
total_sum_df = result_df.groupBy("payer_account_number").agg(
    F.sum("payer_amount").alias("total_transaction_sum_paid_as_payer"),
    F.count("*").alias("as_a_payee_count"), 
    F.sum("payee_amount").alias("as_a_payee_sum_received")
)

# Join with unique account details
unique_account_details = result_df.select("payer_account_number", "account_holder_name", "account_ifsc", "payer_account_type").distinct()

final_df = unique_account_details.join(F.broadcast(final_agg_df), ["payer_account_number", "payer_account_type"], "left_outer") \
                                 .join(F.broadcast(total_sum_df), "payer_account_number", "left_outer") \
                                 .fillna(0)

# Display the result
final_df.show()




























































Transaction Classification Update: Enhanced Conflict Resolution Logic

The previously utilized script for transaction classification leveraged two distinct frameworks: the Remarks Framework and the Beneficiary Framework. A salient challenge arose when these frameworks produced conflicting categorizations for a singular transaction. The original resolution logic was contingent upon the sequence (seq) in which these frameworks were executed:

seq 'rb': The Remarks framework was prioritized, followed by the Beneficiary framework.
seq 'br': The Beneficiary framework took precedence, succeeded by the Remarks framework.
In scenarios where:

The premier framework ascertained a default category and the latter discerned a specific category, the specific category from the second framework was adopted.
Both frameworks identified specific yet divergent categories, the determination from the primary framework (in accordance with the chosen seq) was upheld.
This methodology occasionally led to misclassifications due to an over-reliance on either the Remarks or Beneficiary data.

Enhancement Overview:

Our newly revised script mitigates these classification disparities. The core enhancement is the simultaneous integration of both the Remarks and Beneficiary columns during the classification process. By analyzing the correlation and contextual relationship between both columns, our approach ensures a more nuanced and accurate transaction classification.

Benefits of the Updated Script:

Mitigation of misclassifications.
Superior transaction categorization by comprehending the intricate relationships between the Remarks and Beneficiary data.
Enhanced efficiency in transaction categorization.
We are confident that this advanced conflict resolution logic will provide more precise and reliable results, further enhancing the integrity of the transaction classification process.









# coding: utf-8

# In[1]:

import pandas as pd
from nltk.util import skipgrams

import pyspark
from pyspark.context import SparkContext
from pyspark.sql import SQLContext, HiveContext
from pyspark.storagelevel import StorageLevel
from pyspark.sql.functions import udf
from pyspark.sql.types import *
import pyspark.sql.functions as F
import pyspark.sql.types as T
from pyspark.sql.functions import udf
from pyspark.sql import *
from pyspark.ml import feature as MF
from dateutil import relativedelta
import datetime
# In[2]:



# In[62]:


# In[ ]:

#rtgs_data_acct_ind.count()


# In[ ]:


sc = SparkContext()
sc.setCheckpointDir('/tmp/spark-code-rtgs')
try:
    # Try to access HiveConf, it will raise exception if Hive is not added
    sc._jvm.org.apache.hadoop.hive.conf.HiveConf()
    sqlContext = HiveContext(sc)
    sqlContext.setConf("hive.exec.dynamic.partition", "true")
    sqlContext.setConf("hive.exec.dynamic.partition.mode", "nonstrict")
except py4j.protocol.Py4JError:
    sqlContext = SQLContext(sc)
except TypeError:
    sqlContext = SQLContext(sc)



# In[2]:


t1 ='db_smith.smth_pool_base_rtgs'
t2='db_stage.stg_fle_category_master'


# In[57]:

import ConfigParser
import sys
configFile = sys.argv[1]
#configFile = '/data/08/notebooks/tmp/Anisha/TransactionClassification/smth_pool_rtgs_20190222200554_python.ini'
config = ConfigParser.ConfigParser()
config.read(configFile)


# In[58]:

data_dtt = config.get('default', 'MASTER_DATA_DATE_KEY').replace("'",'').replace('"','')
t1 = config.get('default','INP_DB_NM_1') + '.' +      config.get('default','INP_TBL_NM_1')

t2 = config.get('default', 'INP_DB_NM_2') + '.' +      config.get('default','INP_TBL_NM_2')
t2_batch = config.get('default', 'END_BATCH_ID_2')

output_tbl=config.get('default', 'OUT_DB_NM') + '.' +             config.get('default','OUT_TBL_NM')

#data_dtt='2019-08-19'


# In[14]:

rtgs_data=(sqlContext.table(t1).filter((F.col('data_dt')>=( datetime.datetime.strptime(data_dtt, "%Y-%m-%d")-relativedelta.relativedelta(days=7)))
											#&(F.col('data_dt')<=str(data_dtt))
											).drop('benef_id','self_flag'))

category_master=sqlContext.table(t2)


# In[15]:

def replaceNull(df, col_list,default_value=''):
    for col in col_list:
        df = df.withColumn(col,F.when(F.col(col).isNull(),default_value).otherwise(F.col(col)))
    return df

col_list1 = ['base_txn_text','benef_nickname','rmtr_to_bene_note']
rtgs_data1 = replaceNull(rtgs_data,col_list1)


# In[16]:

rtgs_data2=rtgs_data1.withColumn('Remarks',(F.upper(F.concat((F.regexp_replace(F.col('base_txn_text'),'(\d+)','')),
                                                           #F.lit(' '),
                                                           #(F.regexp_replace(F.col('derived_txn_txt'),'(\d+)','')),
                                                           F.lit(' '),
                                                           (F.regexp_replace(F.col('rmtr_to_bene_note'),'(\d+)',''))))))


# In[17]:

rtgs_data3=rtgs_data2.withColumn('Remarks1',F.concat(F.col('Remarks'),F.lit(" "),F.col('benef_nickname')))


# In[18]:

root_path = '/ybl/dwh/artifacts/sherlock/pythonJobs'
#root_path='.'
sc.addPyFile(root_path + '/Transaction-Classification/EntityFW.py')
from EntityFW import *
kpp, regex_list = initilize_keywords(root_path,sc,['NACH', 'crowdsource', 'DD', 'Cheques', 'COMMON','TRANSFERS','Ecollect'])
sc.addPyFile(root_path +'/Transaction-Classification/RemarksFW.py')
from RemarksFW import *
R_kp, R_result_dict = R_initialize(root_path,sc)
sc.addPyFile(root_path +'/Transaction-Classification/RemarkEntityWrapper.py')
from RemarkEntityWrapper import *

df_res = ApplyFWSequence(root_path,sc,rtgs_data3,'benef_name','Remarks1', 'category_code', 'benef_id',R_kp, R_result_dict,kpp,
                         regex_list,'rb','510000', remit_col='remitter_name', self_tnfr_col='self_flag')


# In[19]:

purpose_code=sc.parallelize([('PC01','410000'),
('PC02','410000'),
('PC03','110000'),
('PC04','110000'),
('PC05','350200'),
('PC06','190000'),
('PC07','390000'),
('PC08','290400'),
('PC09','290000'),
('PC10','230000'),
('PC11','150000'),
('PC12','160000'),
('PC13','120001'),
('PC31','410000')]).toDF(['code','category_code1'])


# In[20]:

df_res1=df_res.join(F.broadcast(purpose_code),'code','left')


# In[21]:

df_res2=df_res1.withColumn('category_code',F.when((F.col('category_code').isin('510000'))
                                                  &(~F.col('category_code1').isNull()),F.col('category_code1'))
                                                                                       .otherwise(F.col('category_code')))


# In[22]:

df_res3=df_res2.withColumn('benef_name',F.when((F.col('benef_id').isNull()|F.col('benef_id').isin('')),F.col('benef_name')).otherwise(F.col('benef_id')))


# In[23]:

rtgs_final=df_res3.join(category_master,'category_code','left')


# In[7]:

rtgs_all1=rtgs_final.select(F.col("txn_ref_no"),
F.col("txn_date"),
F.col("txn_amt"),
F.col("mode"),
F.col("remitter_id"),
F.col("remitter_name"),
F.col("remitter_type"),
F.col("remitter_class"),
F.col("remitter_sub_class"),
F.col("remitter_ifsc"),
F.col("remitter_bank_name"),
F.col("remitter_account_no"),
F.col("remitter_cust_id"),
F.col("benef_id"),
F.col("benef_name"),
F.col("benef_type"),
F.col("benef_class"),
F.col("benef_sub_class"),
F.col("benef_ifsc"),
F.col("benef_bank_name"),
F.col("benef_account_no"),
F.col("benef_cust_id"),
F.col("base_txn_text"),
F.col("rmtr_to_bene_note"),
F.col("online_offline"),
F.col("category_level1"),
F.col("category_level2"),
F.col("category_level3"),
F.col("category_code"),
F.col("recurrance_flag"),
F.col("recurrance_pattern_id"),
F.col("verification_flag"),
F.col("self_flag"),
F.col("txn_ref_key"),
F.col("channel_code"),
F.col("codstatus"),
F.col("acctstatus"),
F.col("msgstatus"),
F.col("codcurr"),
F.col("datvalue"),
F.col("direction"),
F.col("msgtype"),
F.col("submsgtype"),
F.col("benef_nickname"),
F.col("utr"),
F.col("iduserreference"),
F.col("idrelatedref"),
F.col("channel"),
F.col("I_C"),
F.col('txntype'),
F.col('data_dt'))




# In[109]:

output_cols = sqlContext.table(output_tbl).columns



to_fill = rtgs_all1.columns




res = rtgs_all1
for x in output_cols:
    if x  not in to_fill:
        res = res.withColumn(x, F.lit(''))



res_to_write = res.select(output_cols)

res_to_write.write.insertInto(output_tbl, True)
-----------------------------------------------------------------------
pyRemarksFW

import os, io, re, csv
import string
from collections import Counter
from itertools import permutations,chain
import time
import ConfigParser
from textutils.viktext import KeywordProcessor

class RemarksFW():
    def __init__(self, kp_b, result_dict_b, default_cat):
        self.kp_b = kp_b
        self.result_dict_b = result_dict_b
        self.default_cat = default_cat
        
    def get_branch(self, start, end, startswith, category_set, level):
        rem = ''
        c2 = Counter([x[start:end] for x in category_set if x.startswith(startswith) and x[start:end] != '00'])
        level2 = [x[0] for x in c2.most_common(5)]
        if len(level2) == 1:
            l2 = level2[0]
        elif len(level2) == 0:
            l2 = '00'
        else:
            l2 = '00'
            rem += 'Conflict at level ' + level + ' '
        return l2, rem

    def resolve_deeper(self, start, end, category_set):
    #     print start, end, category_set[0][start:end], category_set[1][start:end]
        if category_set[0][start:end] != '00' and category_set[1][start:end] == '00':
            return category_set[0]
        elif category_set[0][start:end] == '00' and category_set[1][start:end] != '00':
            return category_set[1]


    def get_deepest_category_among_two(self, category_set):
        res = self.resolve_deeper(4,6, category_set)
        if res is None:
            res = self.resolve_deeper(2,4, category_set)
        return res

    
    def conflict_resolver(self, category_set):
        category_set = list(category_set)

        if len(category_set) == 0:
            return self.default_cat, "No match Found"
        else:
            common_two = Counter(category_set).most_common(2)
            if len(common_two) == 2 and common_two[0][1] == common_two[1][1] and common_two[0][0][:2] != common_two[1][0][:2]:
                category_set = [x[0] for x in common_two]
                category = self.get_deepest_category_among_two(category_set)
                if category:
                    return category, "Resolved to deeper between two different primary categories"
                else:
                    return self.default_cat, 'Conflict couldnot be resolved'
            else:    
                remark = ''
                c = Counter([x[:2] for x in category_set])
                l1 = c.most_common(1)[0][0]
                category_set = list(set(category_set))
                l2, rem = self.get_branch(2, 4, l1, category_set, '2')
                remark += rem
                l3, rem = self.get_branch(4, 6, l1+l2, category_set, '3')
                remark += rem
                return l1+l2+l3, remark.strip()
    """
    def registerudf(self):
        return F.udf(self.main_remarks_category, T.StringType())
    """
    def main_remarks_category_old(self, remark):
        try:
            if remark:
                words=self.kp_b.value.extract_keywords(remark)
                words_set = set(words)
                res = []
                for ele in words_set:
                    for set1 in self.result_dict_b.value[ele]:
                        if ((words_set >= set(set1))):
                            res.append(self.result_dict_b.value[ele][set1])
                r = self.conflict_resolver(res)[0]
                return r
            else:
                return self.default_cat
        except:
            return self.default_cat
    
    def main_remarks_category(self, remark):        
        if remark:
            words=self.kp_b.value.extract_keywords(remark)
            words_set = set(words)
            res = []
            for ele in words_set:
                for set1 in self.result_dict_b.value[ele]:
                    if ((words_set >= set(set1))):
                        res.append(self.result_dict_b.value[ele][set1])
            #print 'Matched Categories-', res
            if len(res) == 0:
                return self.default_cat
            res_res = Counter(res).most_common()
            if len(res_res) == 1:
                #print 'after conflict resolution-', res_res[0][0]
                return res_res[0][0]
            
            elif len(res_res) == 2 and res_res[0][0].startswith('13') and not res_res[1][0].startswith('13'):
                return res_res[1][0]

            elif len(res_res) == 2 and res_res[1][0].startswith('13') and not res_res[0][0].startswith('13'):
                return res_res[0][0]
                
            elif Counter(res).most_common()[0][1] > Counter(res).most_common()[1][1]*2:
                #print 'after conflict resolution-', res_res[0][0]
                return res_res[0][0]
            else:
                r = self.conflict_resolver(res)[0]
                #print 'after conflict resolution-', r
                return r
        else:
            return self.default_cat
    
        
    def main_remarks_category_tester(self, remark):        
        if remark:
            words=self.kp_b.extract_keywords(remark)
            words_set = set(words)
            res = []
            for ele in words_set:
                for set1 in self.result_dict_b[ele]:
                    if ((words_set >= set(set1))):
                        res.append(self.result_dict_b[ele][set1])
            #print 'Matched Categories-', res
            if len(res) == 0:
                print self.default_cat
                return self.default_cat
            res_res = Counter(res).most_common()
            if len(res_res) == 1:
                #print 'after conflict resolution-', res_res[0][0]
                return res_res[0][0]
            elif Counter(res).most_common()[0][1] > Counter(res).most_common()[1][1]*2:
                #print 'after conflict resolution-', res_res[0][0]
                return res_res[0][0]
            else:
                r = self.conflict_resolver(res)[0]
                #print 'after conflict resolution-', r
                return r
        else:
            return self.default_cat

            
def R_combine(a_list):
    res = []
    for i, ele in enumerate(a_list):
        k = ''.join(a_list[:i]) + ' ' + ''.join(a_list[i:])
        res.append(k.strip()) 
    return res

def R_combine2(a_list):
    res = []
    for i, ele in enumerate(a_list):
        k = ''.join(a_list[:i]) + ' ' + ''.join(a_list[i:])
        k2 = ''.join(a_list[:i])  + ' '+ ''.join(a_list[i:]) +'s'
        res.append(k.strip())
        res.append(k2.strip()) 
    return res 


    
def R_get_keywords_from_csv(filename):
    result_dict = {}
    cat_key_mapp={}
    from itertools import permutations,chain
    with open(filename, 'rb') as filereader:
        rd = csv.reader(filereader)
        for line in rd:
            code = line[3]
            keywords = line[4].lower().replace('[','') \
                              .replace(']','').replace("'","").replace('0', '').replace('\n', '').strip().split(',')
            cat_key_mapp[code]=(line[1]+" "+line[2]).strip()
            for ele in keywords:
                if len(ele.strip()) > 1:
                    if len(ele.split()) > 2:
                        k2 = [R_combine(x) for x in list(permutations(ele.split()))]
                        merged = list(chain(*k2))
                        merged.append(ele)
                    else:
                        k2 = [R_combine2(x) for x in list(permutations(ele.split()))]
                        merged = set(list(chain(*k2)))
                    for elem in merged:    
                        k1 = elem.split(' ')
                        for k11 in k1:
                            if k11 != '':
                                k11.strip()
                                if k11 not in result_dict:
                                    result_dict[k11] = {}
                                result_dict[k11][tuple(elem.split())] = code
    return result_dict

def R_initialize(root_path, fname = '/Transaction-Classification/MasterData/Txn_Classification_28March.csv'):
    """
    root_path -> directory where Transaction-Classification folder is present
    sc -> spark context
    fname -> path to csv containing remarks master, default: './Transaction-Classification/MasterData/Txn_Classification_28March.csv'
    """
    result_dict = R_get_keywords_from_csv(root_path + fname)
    kp = KeywordProcessor()
    kp.add_keywords_from_list([k for k in result_dict])
    return kp, result_dict
-------------------------------------------------------------------------------
pyEntityFW

import csv
import re

def pycleaner(text, regex_list):
    for reg in regex_list:
        text = re.sub(reg, ' ', text)
    return text.lower().strip()


def get_one_to_many_category_map2(file_name, filters, clean_regex_list):
    one_many_map = {}
    with open(file_name, 'rb') as csvfile:
        spamreader = csv.reader(csvfile)
        for i, row in enumerate(spamreader):
            if i == 0:
                row = [x.upper() for x in row]
                entity_index = row.index('ENTITY_ID')
                keyword_index = row.index('KEY')
                cat_code_index = row.index('CATEGORY_CODE')
                channel_key = row.index('CHANNEL')
            
            elif row[channel_key] in filters:
                k1 = pycleaner(row[keyword_index], clean_regex_list)
                k2 = pycleaner(row[entity_index], clean_regex_list)
                channel = row[channel_key]
                v = row[cat_code_index]
                if k2 not in one_many_map:
                    one_many_map[k2] = []
                one_many_map[k2].append(v)
    
    conflict_dict = {}
    for ele in one_many_map:
        if len(list(set(one_many_map[ele]))) > 1:
            conflict_dict[ele] = list(set(one_many_map[ele]))

    return conflict_dict            
    

def get_specific_mapper(file_name, filters, clean_regex_list):
    mapper_dict = {}
    regex_list = []
    conflict_dict = get_one_to_many_category_map2(file_name, filters, clean_regex_list)
    with open(file_name, 'rb') as csvfile:
        spamreader = csv.reader(csvfile)
        for i, row in enumerate(spamreader):
            if i == 0:
                row = [x.upper() for x in row]
                entity_index = row.index('ENTITY_ID')
                keyword_index = row.index('KEY')
                cat_code_index = row.index('CATEGORY_CODE')
                channel_key = row.index('CHANNEL')
            
            elif row[channel_key] in filters:
                v11 = None
                v1 = row[keyword_index]
                if '@' in v1:
                    v11 = v1.split('@')[0]
                
                v2 = pycleaner(row[entity_index].lower(), clean_regex_list)
                if v1.startswith('REGEX::'):
                    v1 = v1.split('::')[1]
                    regex_list.append([v1, row[cat_code_index] +'|'+ row[entity_index].lower()])
                else:
                    v1 = pycleaner(v1.lower(), clean_regex_list)
                    
                    k = row[cat_code_index] + '|' + row[entity_index].lower()
                    if k not in mapper_dict:
                        mapper_dict[k] = []
                    if v1 not in mapper_dict[k] and len(v1) > 1:
                        mapper_dict[k].append(v1)
                        v1_s = v1.replace(' ','')
                        if v1_s != v1:
                            mapper_dict[k].append(v1_s)
                        
                    if v11:
                        v11 = pycleaner(v11.lower(), clean_regex_list)
                        if v11 not in mapper_dict[k] and len(v11) > 1 and v11 not in conflict_dict:
                            mapper_dict[k].append(v11)
                            v11_s = v11.replace(' ', '')
                            if v11_s != v11:
                                mapper_dict[k].append(v11_s)
                    
                    if v2 not in mapper_dict[k] and len(v2) > 1 and v2 not in conflict_dict:
                        mapper_dict[k].append(v2)
                        v2_s = v2.replace(' ','')
                        if v2_s != v2:
                            mapper_dict[k].append(v2_s)
                        
    return mapper_dict, regex_list

def initilize_keywords(root_path, data_list, regex_list= [r'[^A-Za-z\&]+', r'\bNULL\b', r'\s+'], data_csv= '/MasterData/FINAL_MAPPER_DATA.csv'):
    """
    root_path -> directory where Transaction-Classification folder is present
    sc -> spark context
    data_list -> list of beneficiary name datasets from the mapper table, contains these types at max: ['EPI', 'POS', 'crowdsource', 'DD', 'Cheques', 'UPI', 'NACH']
    regex_list -> list of regular expressions for cleanup, default vaule: [r'[^A-Za-z\&]+', r'\bNULL\b', r'\s+']
    data_csv -> path to mapper file in csv format, default_value: './Transaction-Classification/MasterData/FINAL_MAPPER_DATA.csv'
    """
    #sc.addPyFile(root_path + '/Transaction-Classification/textutils/viktext.py')
    from textutils.viktext import KeywordProcessor
    data_csv = root_path + data_csv
    
    res, regex_list = get_specific_mapper(data_csv, data_list, regex_list)
    kpp = KeywordProcessor()
    kpp.add_keywords_from_dict(res)
    return kpp, regex_list
'''
def extractRegex2(default, benif_col, regex_list,  i = 0):
    if i == len(regex_list):
        return default
    else:
        return F.when(benif_col.rlike(regex_list[i][0]),regex_list[i][1]) \
                .otherwise(extractRegex2(default, benif_col, regex_list, i = i+1))
        
def textcleaner2(df, col_name, regex_list = [r'[^A-Za-z\&]+', r'\bNULL\b', r'\s+'], i = 0):
    if i == len(regex_list):
        return df.withColumn('_Benif_clean_', F.lower(F.trim(col_name)))    
    else:
        funct = F.regexp_replace(col_name, regex_list[i], ' ')
        return textcleaner2(df, col_name = funct, i = i+1)
    
def process_beneficiary(df, name_col, cat_col, entity_col, regex_list, kp_b, default_cat = '510000', cleaner_regex_list=[r'[^A-Za-z\&]+', r'\bNULL\b', r'\s+']):
    """
    df -> Input dataframe
    name_col -> column name containing beneficiary name
    cat_col -> output column name which would contain category code corresponing to the beneficiary name passed
    entity_col -> output column name which would contain entity id, which is currently the normalized entity name
    regex_list -> list of list of regex element1 is regex and element2 is category|
    kp_b -> Broadcast to keywordprocesser object containg mapping keywords
    default_cat -> Category to be passed in case entity not found in the list
    cleaner_regex_list -> list of regular expressions which would be eventually replaced by ' ' character, default_value: [r'[^A-Za-z\&]+', r'\bNULL\b', r'\s+']
    """
    temp_cols = ['_Benif_clean_', '_cat_benif1_', '_cat_benif2_', '_cat_code_', '_entity_id_']
    df2 = textcleaner2(df, F.col(name_col), cleaner_regex_list)
    df2 = df2.fillna('NA', subset=['_Benif_clean_'])
    
    df3 = df2.select('_Benif_clean_').dropDuplicates(['_Benif_clean_'])
    
    r = benifUDF(kp_b, default_cat)
    main_benif_categoryUDF = r.registerudf()
    
    #main_benif_categoryUDF = F.udf(main_benif_category, T.StringType())
    res = df3.withColumn('_cat_benif1_', main_benif_categoryUDF(F.col('_Benif_clean_')))
    
    if len(regex_list) > 0:
        res2 = res.withColumn('_cat_benif2_' ,extractRegex2(F.col('_cat_benif1_'), F.col('_Benif_clean_'), regex_list))
        split_col=F.split(res2['_cat_benif2_'],'\|')
        df4 = res2.withColumn("_cat_code_",split_col.getItem(0)).withColumn('_entity_id_', split_col.getItem(1)) \
            .select(['_Benif_clean_',"_cat_code_", '_entity_id_'])
    else:
        res2 = res
        split_col=F.split(res2['_cat_benif1_'],'\|')
        df4 = res2.withColumn("_cat_code_",split_col.getItem(0)).withColumn('_entity_id_', split_col.getItem(1)) \
            .select(['_Benif_clean_',"_cat_code_", '_entity_id_'])
    df4 = df4.withColumnRenamed('_entity_id_', entity_col).withColumnRenamed('_cat_code_', cat_col)
    df_res = df2.join(df4, '_Benif_clean_', 'left')
    return df_res.drop(F.col('_Benif_clean_'))
'''

class benifUDF:
    def __init__(self, kp_b, default_cat):
        self.kp_b = kp_b
        self.default_cat = default_cat
    '''    
    def registerudf(self):
        return F.udf(self.main_benif_category, T.StringType())
    '''
    def main_benif_category(self, benif_name):
        """
        UDF to find keywords presence
        depends upon global broadcase variable kp_b
        """
        words = self.kp_b.extract_keywords(benif_name)
        if len(words) > 0:
            return words[0]
        else:
            return self.default_cat +'|' 
--------------------------------------------------------------------------------
Remark Entity Wrapper- 

import pyspark.sql.types as T
import pyspark.sql.functions as F


def ApplyFWSequence(root_path, sc, df, benif_col, remark_col, 
                         out_catagory_col, out_entity_id, 
                         R_kp, R_result_dict, kpp, regex_list, seq='rb', default_cat = '170000', 
                         remit_col=None, self_tnfr_col=None, broadcasting=False):
    """
    root_path -> path where Transaction-Classification folder is present
    sc -> Spark Context
    df -> Input transactions dataframe
    remark_col -> Column name containing remarks
    out_catagory_col -> output column name for category code
    out_entity_id -> output column name for entity id (which currently is same as entity name)
    R_kp -> Knowledge Processor object for remarks Framework
    R_result_dict -> Remarks Result dictonary
    kpp -> Knowledge Processor object for Entity Framework (ouput of initilization)
    regex_list -> regex list (output of initilization of Entity FW)
    seq -> 'rb' or 'br' (r: remarks FW & b: beneficiary FW)
    default_cat -> default category code
    remit_col -> remitter column name
    self_tnfr_col -> output column which would contain self transfer flag (0/1)
    """
    sc.addPyFile(root_path + '/Transaction-Classification/EntityFW.py')
    sc.addPyFile(root_path + '/Transaction-Classification/RemarksFW.py')
    from RemarksFW import *
    from EntityFW import *
    
    df_res = None
    temp_cat_col = 'T_' + out_catagory_col
    
    for char in seq:
        if char == 'r':
            if not df_res: 
                #df_res = R_get_txn_class_remark(sc, df_res, 'base_txn_text', 'category_code_FW', R_kp, R_result_dict, default_cat)
                df_res = R_get_txn_class_remark(root_path, sc, df, remark_col, out_catagory_col, R_kp, R_result_dict, 
                                                default_cat, remit_col, benif_col, self_tnfr_col, broadcasting)
            else:
                df_res2 = R_get_txn_class_remark(root_path, sc, df_res, remark_col, temp_cat_col, R_kp, R_result_dict,
                                                 default_cat,  remit_col, benif_col, self_tnfr_col, broadcasting)
                
                
        if char == 'b':
            if not df_res:
                #df_res = process_beneficiary(df, 'benef_name', 'benif_cat', 'benif_id', regex_list, kpp, default_cat)
                df_res = process_beneficiary(df, benif_col, out_catagory_col, out_entity_id, regex_list, kpp, default_cat, broadcasting = broadcasting)
            else:
                df_res2 = process_beneficiary(df_res, benif_col, temp_cat_col, out_entity_id, regex_list, kpp, default_cat, broadcasting = broadcasting)
    
    df_res3 = df_res2.withColumn(out_catagory_col,
                                 F.when(F.col(out_catagory_col) == default_cat,
                                        F.col(temp_cat_col)) \
                                 .otherwise(F.col(out_catagory_col))
                                ).drop(temp_cat_col)
    return df_res3
