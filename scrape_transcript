import time
import re
from selenium import webdriver
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np
import os

class Pathe:
    def __init__(self,name):
        self.name = name
        self.nums = 0 
class Participants:
    def __init__(self):
        self.positionList = []
        
class Personel():
    def __init__(self,intro):
        self.intro = intro
        self.name = intro.split("-")[0]
        self.position = intro.split("-")[1]
        
pd.set_option('display.max_rows', None)
pd.set_option('display.max_rows', None)

try:
    os.mkdir(r"D:\JFE")
except:
    print("folder exists")

url_list = []
date_time = []
headlines = []
tcpt_text = []
QA = []#****** changed *****#
Company_Name_sub = []
Company_Symbol_sub = []
Company_Name = [] #name from the bold title
Company_Symbol = [] #symbol from the bold title
fisc_date_head = []
fisc_date = [] #fiscal date from the bold title
u = []
numj = 0
Comp_part = []
Conf_part = []
Comp_position = []
Conf_position = []

driver = webdriver.Chrome(executable_path = r"D:\chromedriver-win32\chromedriver.exe")
driver.get("https://seekingalpha.com/earnings/earnings-call-transcripts")
numi = 0
for i in range(6184): # 6184, select pages range ~2004
    url = driver.current_url + "?page={i}".format(i = i)
    request = requests.get(url)
    soup = BeautifulSoup(request.text,"html.parser")
    for info in soup.find_all("a",{"class":"jC_R"}): #****1****#
        if info.get("href") not in url_list:
            url_list.append(info.get("href"))
    numi += 1
    print("{} page has finished".format(numi-1),end = '\r')
print("{} urls in total".format(len(url_list)))
for i in url_list:
    i = "https://seekingalpha.com"+i
    u.append(i)
u = pd.DataFrame({"urls":u})
u.to_csv(r"D:\JFE\urllist.csv")
del u

urlList_pre = pd.read_csv(r"D:\JFE\urllist.csv")
urlList = urlList_pre.urls.to_list()
uslice = urlList[128500:129500]# 247,311 in total
del urlList
headers = {'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36'}
for j in uslice:
    #URL=>headlines
    s = j.split("/")[-1].replace("-"," ")
    headlines.append(s)
    #catch the page element from seekalpha 
    request = requests.get(j,headers = headers)
    request.close()
    soup = BeautifulSoup(request.text, "html.parser")
    # extract Company Name and Company symbol from subtitle
    if soup.find_all("span",{"data-test-id":"post-primary-tickers"}) != []: #****2****#
        for names in soup.find_all("span",{"data-test-id":"post-primary-tickers"}):
            for name in names:
                a = name.text
                Company_Name_sub.append(a)
                b = re.findall(r"[(]([A-Z]+)[)]",a)     
                try:
                    b0 = b[0]
                    Company_Symbol_sub.append(b0)
                except:
                    b0 = 0
                    Company_Symbol_sub.append(b0)
                break             
    else:
        Company_Name_sub.append("0")
        Company_Symbol_sub.append("0")
    # probe date time in the subtitle/navigator 
    if soup.find_all("span",{"data-test-id":"post-date"}) != []: #****3****#
        for dates in soup.find_all("span",{"data-test-id":"post-date"}):
            date_time.append(dates.text)
            break            
    else:
        date_time.append("0")
    #main body => tcpt_text => "details"   
    temp1 = []# ***** changed ***** #
    temp2 = []# ***** changed ***** #
    num_tag = 0
    for content in soup.find_all("div",{"data-test-id":"content-container"}):#****4****## ***** changed ***** #
        try:
            for tags in content.find_all("p"):
                try:
                    tags["id"] == "question-answer-session"
                    break
                except:
                    num_tag += 1
            for i in range(num_tag):
                temp1.append(content.find_all("p")[i].text)
            for i in range(num_tag,len(content.find_all("p"))):
                temp2.append(content.find_all("p")[i].text)
        except:
            pass

    joined_text1 = " ".join(temp1)# ***** changed ***** #
    tcpt_text.append(joined_text1)# ***** changed ***** #
    joined_text2 = " ".join(temp2)
    QA.append(joined_text2)# ***** changed ***** #
    
    try:
        num_comp_part = temp1.index("Company Participants")
    except:
        num_comp_part = "nan"
    try:    
        num_conf_part = temp1.index("Conference Call Participants")
    except:
        num_conf_part = "nan"
    try:
        num_op = temp1.index("Operator")
    except:
        num_op = "nan"
        
    Part_Company = Participants()
    Part_Conference = Participants()
    temp3 = []
    temp4 = []
    try:
        for i in range(num_comp_part+1,num_conf_part):
            ptcp = Personel(temp1[i])
            temp3.append(ptcp.intro)
            Part_Company.positionList.append(ptcp.position)
        joined_text3 = " ".join(temp3)
        Company_position = ",".join(Part_Company.positionList)
    except:
        joined_text3 = ""
        Company_position = ""
    
    try:
        for i in range(num_conf_part+1,num_op):
            ptcp = Personel(temp1[i])
            temp4.append(ptcp.intro)
            Part_Conference.positionList.append(ptcp.position)
        joined_text4 = " ".join(temp4)
        Conference_position = ",".join(Part_Conference.positionList)   
    except:
        joined_text4 = ""
        Conference_position = ""
    
    Comp_part.append(joined_text3)
    Conf_part.append(joined_text4)
    Comp_position.append(Company_position)
    Conf_position.append(Conference_position)
    
    numj += 1
    print("{} url has finished".format(numj),end = '\r')
    #bold title =>company name/symbol & fiscal quarter/year
    if soup.find_all("h1",{"data-test-id":"post-title"}) != []:#****5****#
        for name in soup.find_all("h1",{"data-test-id":"post-title"}):
            c = name.text
            Company_Name.append(c)
            d = re.findall(r"[(]([A-Z]+)[)]",c)
            try:
                d0 = d[0]
                Company_Symbol.append(d0)
            except:
                d0 = 0
                Company_Symbol.append(d0)
            try:
                s = re.search("[Qq]\d \d{4}",c)
                fisc_date.append(s.group())
            except:
                fisc_date.append("N/A")
            break  
    else:
        Company_Name.append("0")
        Company_Symbol.append("0")
        fisc_date.append("N/A")
#headlines=> fisc_date_head    
Title = []
for i in headlines:
    s = i.split(" ",1)[1]
    Title.append(s)
    try:
        s = re.search("[Qq]\d \d{4}",i)
        fisc_date_head.append(s.group())
    except:
        fisc_date_head.append("N/A")
#subtitle =>company_name_sub(New_name_sub)    
New_name_sub = []
for i in range(len(Company_Name_sub)):
    if Company_Name_sub[i] != "0":
        new_name_sub = Company_Name_sub[i].replace(f"({Company_Symbol_sub[i]})","")
        New_name_sub.append(new_name_sub)
    else:
        New_name_sub.append("0")
#bold title => company_name(New_name)       
New_name = []
for i in range(len(Company_Name)):
    if Company_Name[i] != "0":
        try:
            new_name = re.findall(r"(.*)[(]",Company_Name[i])
            New_name.append(new_name[0].strip())
        except:
            New_name.append("0")
    else:
        New_name.append("0")
#headlines => fisc_date_head => Fiscal_Year_head/Fiscal_Month_head   
Fiscal_Year_head = []
Fiscal_Month_head = []
for i in fisc_date_head:
    if i != "N/A":
        Fiscal_Year_head.append(i.split(" ")[1])
        Fiscal_Month_head.append(i.split(" ")[0].upper())
    else:
        Fiscal_Year_head.append("na")
        Fiscal_Month_head.append("na")
#bold title => fisc_date => Fiscal_Year/Fiscal_Month
Fiscal_Year = []
Fiscal_Month = []
for i in fisc_date:
    if i != "N/A":
        Fiscal_Year.append(i.split(" ")[1])
        Fiscal_Month.append(i.split(" ")[0].upper())
    else:
        Fiscal_Year.append("na")
        Fiscal_Month.append("na")
#subtitle navigator => date_time => report_MDYT
report_month = []
report_day = []
report_year = []
report_time = []
for i in date_time:
    if i != "0":
        m = re.search(".{3}",i)
        report_month.append(m.group())
        d = re.search("[. ]\d{2}",i)
        report_day.append(d.group())
        y = re.search("[, ]\d{4}",i)
        report_year.append(y.group())
        t = i.split(" ",3)[-1]
        report_time.append(re.sub("ET","",t))
    else:
        report_month.append("N/A")
        report_day.append("N/A")
        report_year.append("N/A")
        report_time.append("N/A")
# summary as df                
df = pd.DataFrame({"Company Name Sub":New_name_sub,"Company Ticker Sub":Company_Symbol_sub,"Company Name":New_name,
                   "Company Ticker":Company_Symbol,"Fiscal Year Head":Fiscal_Year_head,"Fiscal Quarter Head":Fiscal_Month_head,
                   "Fiscal Year":Fiscal_Year,"Fiscal Quarter":Fiscal_Month,"Report Month":report_month,
                   "Report Day":report_day,"Report Year":report_year,"Report Time":report_time,
                   "Title":Title,"details":tcpt_text,"QA":QA,"Company participants":Comp_part,
                   "Conference participants":Conf_part,"Company positions":Comp_position,"Conference positions":Conf_position,
                   "call link":uslice})
df.to_csv(r"D:\JFE\record1.csv")

del df,url_list,date_time,headlines,tcpt_text,Company_Name_sub,Company_Symbol_sub,Company_Name,Company_Symbol,fisc_date_head,fisc_date
df = pd.read_csv(r"D:\JFE\record1.csv")
#modify the type of certain columns
df["Report Year"]=df["Report Year"].map(str)
ry = df["Report Year"].tolist()
for i in range(len(ry)):
    ry[i] = ry[i].split(".")[0]
df["Report Year"] = ry
df["Report Day"]=df["Report Day"].map(str)
rd = df["Report Day"].tolist()
for i in range(len(rd)):
    rd[i] = rd[i].split(".")[0]
df["Report Day"] = rd

#cleanse data
df = df.drop(columns = ["Unnamed: 0"])
import numpy as np
df["details"] = df["details"].fillna("nan")
df["Report Month"] = df["Report Month"].fillna("nan")
df["details"]=df["details"].map(str)
df["Company Ticker"]=df["Company Ticker"].map(str)
df["Company Ticker Sub"]=df["Company Ticker"].map(str)
df = df.drop(index = df[df["Company Name"] == "0"].index.to_list())
df = df.drop(index = df[df["Company Ticker"] == "0"].index.to_list())
df = df.drop(index = df[df["Fiscal Year"] == "na"].index.to_list())
df = df.drop(index = df[df["Fiscal Quarter"] == "na"].index.to_list())
df = df.reset_index()
df.to_csv(r"D:\JFE\record2.csv")

# #summary the missing values

# df = df.drop(columns = ["Unnamed: 0"])
# import numpy as np
# df["details"] = df["details"].fillna("nan")
# df["Report Month"] = df["Report Month"].fillna("nan")
# df["details"]=df["details"].map(str)
# df["Company Ticker"]=df["Company Ticker"].map(str)
# df["Company Ticker Sub"]=df["Company Ticker"].map(str)

# mvs = ["0","0","0","0","na","na","na","na","nan","nan","nan","nan","nan","nan",0,"na"]
# for i in range(len(df.columns)):
               
#                print("missing values counts in {0} is {1}".format(df.columns[i],
#                                                                   len(df[df[df.columns[i]] == mvs[i]])))

df = pd.read_csv(r"D:\JFE\record2.csv")
df = df.drop(columns = ["Unnamed: 0","index"])
# add two new columns "Repyq" and "Fisyq", concatenate two parts into "YYYYQn" format.
Q1 = ["Jan","Feb","Mar"]
Q2 = ["Apr","May","Jun"]
Q3 = ["Jul","Aug","Sep"]
Q4 = ["Oct","Nov","Dec"]
Report_Calendar_Quarter = []
for i in range(len(df)):
    if df["Report Month"][i] in Q1:
        Report_Calendar_Quarter.append("Q1")
    elif df["Report Month"][i] in Q2:
        Report_Calendar_Quarter.append("Q2")
    elif df["Report Month"][i] in Q3:
        Report_Calendar_Quarter.append("Q3")
    elif df["Report Month"][i] in Q4:
        Report_Calendar_Quarter.append("Q4")
    else:
        Report_Calendar_Quarter.append(np.nan)
df["Report Quarter"] = Report_Calendar_Quarter
df = df[~(df["Report Quarter"].isna())] #****changed
Repyq = []
for i in range(len(df)):
    s = str(df["Report Year"][i]) + df["Report Quarter"][i]
    Repyq.append(s)
df["Repyq"] = Repyq 
Fisyq = []
for i in range(len(df)):
    s = str(df["Fiscal Year"][i]) + df["Fiscal Quarter"][i]
    Fisyq.append(s)
df["Fisyq"] = Fisyq
#filter out non-regular quarter
quarter = ["Q1","Q2","Q3","Q4"]
flag_quarter = []
for i in range(len(df)):
    if df["Fiscal Quarter"][i] not in quarter:
        flag_quarter.append(1)
    else:
        flag_quarter.append(0)
df["flag_quarter"] = flag_quarter
df = df[df["flag_quarter"] == 0]
df = df.reset_index()
df = df.drop(columns = ["flag_quarter"])
del flag_quarter
#add two new columns "q_month" and "r_month",convert quarter/month in text into numeric format
monthQ = [3,6,9,12]
report_month = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]
monthR = [1,2,3,4,5,6,7,8,9,10,11,12]
q_month = []
r_month = []
for i in range(len(df)):
    q_month.append(monthQ[quarter.index(df["Fiscal Quarter"][i])])
    r_month.append(monthR[report_month.index(df["Report Month"][i])])
df["q_month"] = q_month
df["r_month"] = r_month
#seperate data into two parts df and df_abnormal
df_ab_pre1 = df[df["Fiscal Year"] > df["Report Year"]]
df_ab_pre1 = df_ab_pre1.reset_index()
df_ab_pre1 = df_ab_pre1.drop(columns = ["index"])
df = df[df["Fiscal Year"] <= df["Report Year"]]
df1 = df[df["Fiscal Year"] < df["Report Year"]]
df1 = df1.reset_index()
df2 = df[df["Fiscal Year"] == df["Report Year"]]
df_ab_pre2 = df2[df2["r_month"] <= df2["q_month"]]
df_ab_pre2 = df_ab_pre2.reset_index()
df_ab_pre2 = df_ab_pre2.drop(columns = ["index"])
df2 = df2[df2["r_month"] > df2["q_month"]]
df2 = df2.reset_index()
df = pd.concat([df1,df2])
df_abnormal = pd.concat([df_ab_pre1,df_ab_pre2])
df_abnormal = df_abnormal.drop(columns = ["level_0"])
df_abnormal.to_csv(r"D:\JFE\record_abnormal.csv")
del df_abnormal
#clean the data in df and generate record_normal.csv
df = df.drop(columns = ["level_0","index"])
df = df.reset_index()
df = df.drop(columns = ["index"])
df.to_csv(r"D:\JFE\record_normal.csv")
del df

os.system("echo test>pw.pgpass")
os.system("echo 'wrds-pgdata.wharton.upenn.edu:9737:wrds:kylecao1:**********?' > pw.pgpass")
os.system("attrib +r pw.pgpass")
# !pip install wrds
import wrds
conn = wrds.Connection(wrds_username = "kylecao1")
refe = conn.raw_sql("""select tic, datadate, datacqtr, datafqtr 
                        from comp_na_daily_all.fundq 
                        where datadate>='01/01/2004'""", 
                     date_cols=['date'])

refe.to_csv(r"D:\JFE_check\refe.csv")

def ismatch(a):
    return re.match("\d{4}Q[1-4]",a)
def extract(a):
    a_conv = {"year":re.search("(\d{4})Q([1-4])",a)[1],"quarter":re.search("(\d{4})Q([1-4])",a)[2]}
    return a_conv
def is_prior(a,b):
    if len(a) == 6 and len(b) == 6:
        if ismatch(a) and ismatch(b):
            if extract(a).get("year") < extract(b).get("year"):
                result = True
            elif extract(a).get("year") == extract(b).get("year"):
                if extract(a).get("quarter") <= extract(b).get("quarter"):
                    result = True
                else:
                    result = False
            else:
                result = False
        else:
            result = False
    else:
        result = False
    return result       

df_abnormal = pd.read_csv(r"D:\JFE\record_abnormal.csv")
#modify the column names to merge two table refe and df_abnormal
refe.columns = ["Company Ticker","datadate","datacqtr","Fisyq"]
df_ref = pd.merge(df_abnormal,refe,on = ["Company Ticker","Fisyq"],how = "left" )
df_ref = df_ref.dropna()
#check if there is no missing value in each record
#df_ref.isnull().sum()
df_ref = df_ref.reset_index()
df_ref = df_ref.drop(columns = ["index","Unnamed: 0"])

flag = []
for i in range(len(df_ref)):
    a = is_prior(df_ref["datacqtr"][i],df_ref["Repyq"][i])
    flag.append(a)
df_ref["flag"] = flag
df_ref = df_ref[df_ref["flag"] == True]

df = pd.read_csv(r"D:\JFE\record_normal.csv")
df = df.drop(columns = ["Unnamed: 0"])
df = pd.concat([df,df_ref])
df = df.reset_index()
del df_ref

beg = []
for i in range(len(df)):
    s = df["details"][i].split(".")
    try:
        sf = s[0]+""+s[1]+""+s[2]+""+s[3]+""+s[4]+""+s[5]+""+s[6]
    except:
        sf = "nan"
    beg.append(sf)
df["begin"] = beg
df["Company Ticker"] = df["Company Ticker"].map(str)

replace_ct = 0
flag = []
for i in range(len(df)):
    if df["Company Ticker"][i] in df["begin"][i]:#     if df["Company Ticker"][i].lower()not in df["Title"][i] or df["Company Ticker"][i] not in df["begin"][i]:
        flag.append(0)
    elif df["Company Ticker Sub"][i] in df["begin"][i]:
        df["Company Ticker"][i] = df["Company Ticker Sub"][i]
        replace_ct += 1
        flag.append(0)
    else:
        flag.append(1)
df["flag"] = flag
#print(replace_ct) out:0
#len(df[df["flag"] == 1]) out:159
df_manu_check1 = df[df["flag"] == 1] # manual check tickers
df_manu_check1.to_csv(r"D:\JFE\manucheck_tic.csv")
#clean ticker mismatches
df = df[df["flag"] == 0]
df = df.reset_index()
df = df.drop(columns = ["index"])

suffix = [",",".","Inc","Ltd","Corp","Corporation","Limited"] 
df["Company Name"] = df["Company Name"].map(str)
df["Company Name Sub"] = df["Company Name Sub"].map(str)
cpn = df["Company Name"].to_list()
cpnsub = df["Company Name Sub"].to_list()
for i in range(len(cpn)):
    for j in suffix:
        cpn[i] = cpn[i].replace(j,"")
    cpn[i] = cpn[i].strip()
df["Name"] = cpn
for i in range(len(cpnsub)):
    for j in suffix:
        cpnsub[i] = cpnsub[i].replace(j,"")
    cpnsub[i] = cpnsub[i].strip()
df["Name Sub"] = cpnsub

beg = df["begin"].to_list()
for i in range(len(beg)):
    for j in suffix:
        beg[i] = beg[i].replace(j,"")
    beg[i] = beg[i].strip()
df["beg"] = beg

replace_ct = 0
flag = []
for i in range(len(df)):
    if df["Name"][i] in df["beg"][i]:#     if df["Company Ticker"][i].lower()not in df["Title"][i] or df["Company Ticker"][i] not in df["begin"][i]:
        flag.append(0)
    elif df["Name Sub"][i] in df["beg"][i]:
        df["Company Name"][i] = df["Company Name Sub"][i]
        replace_ct += 1
        flag.append(0)
    else:
        flag.append(1)
df["flag"] = flag
#replace_ct out:9380
#len(df[df["flag"] == 1]) out:1343
#clean name mismatch
df = df[df["flag"] == 0]
df = df.reset_index()
df = df.drop(columns = ["index","Name","Name Sub","beg","flag","datadate","datacqtr","q_month","r_month","Repyq","Fisyq","Report Quarter"])

flag = []
replace_ct = 0
for i in range(len(df)):
    if df["Fiscal Quarter"][i] in df["begin"][i]:
        flag.append(0)
    elif df["Fiscal Quarter Head"][i] in df["begin"][i]:
        df["Fiscal Quarter"][i] = df["Fiscal Quarter Head"][i]
        replace_ct += 1
        flag.append(0)
    else:
        flag.append(1)
df["flag"] = flag
df_manu_check2 = df[df["flag"] == 1] # manual check fiscal quarter
df_manu_check2.to_csv(r"D:\JFE\manucheck_quarter.csv")
df = df[df["flag"] == 0]
df = df.reset_index()
df = df.drop(columns = ["level_0","index","flag"])

df["Fiscal Year"]=df["Fiscal Year"].map(str)
df["Fiscal Year Head"]=df["Fiscal Year Head"].map(str)
flag = []
replace_ct = 0
for i in range(len(df)):
    if df["Fiscal Year"][i] in df["begin"][i]:
        flag.append(0)
    elif df["Fiscal Year Head"][i] in df["begin"][i]:
        df["Fiscal Year"][i] = df["Fiscal Year Head"][i]
        replace_ct += 1
        flag.append(0)
    else:
        flag.append(1)
df["flag"] = flag
#len(df[df["flag"] == 1]) out:40
df_manu_check3 = df[df["flag"] == 1] # manual check fiscal quarter
df_manu_check3.to_csv(r"D:\JFE\manucheck_year.csv")
df = df[df["flag"] == 0]
df = df.reset_index()
df = df.drop(columns = ["index","Company Name Sub","Company Ticker Sub","Fiscal Year Head","Fiscal Quarter Head","begin","flag"])
df1 = df.drop(columns = ["details"])
df1.to_csv(r"D:\JFE\record_all.csv")
#pick out fiscal year later than 2020
df["Fiscal Year"] = df["Fiscal Year"].map(int)
#df = df[df["Fiscal Year"] >= 2020] #!!!!!!*****#
#len(df) out:53831
df = df.reset_index()

os.mkdir(r"D:\JFE\transcript2")
class Pathe:
    def __init__(self,name):
        self.name = name
        self.nums = 0 
l_path = []
obj_path = []

df["QA"] = df["QA"].map(str)

for i in range(len(df)):
    s_path = r"D:\JFE\transcript2\{}_{}_{}.txt".format(df["Company Ticker"][i],df["Fiscal Quarter"][i],df["Fiscal Year"][i])
    if s_path not in l_path:
        l_path.append(s_path)
        path = Pathe(s_path)
        obj_path.append(path)
        
        file = open(s_path,"w+",encoding = "utf-8")
        file.write(df.details[i]+"\n\n\n\n"+df.QA[i])#***** changed *****#
        file.close()
    elif s_path in l_path:
        g = open(s_path)
        text = g.readlines()
        if text[0] != df.details[i]:
            obj_path[l_path.index(s_path)].nums += 1
            count = obj_path[l_path.index(s_path)].nums
            s_path = r"D:\JFE\transcript2\{}_{}_{}_{}.txt".format(df["Company Ticker"][i],df["Fiscal Quarter"][i],df["Fiscal Year"][i],count)
            file = open(s_path,"w+",encoding = "utf-8")
            file.write(df.details[i]+"\n\n\n\n"+df.QA[i])#***** changed *****#
            file.close()
        else:
            continue

df = df.drop(columns = ["index","details"])
df.to_csv(r"D:\JFE\record_Jul20.csv")
# count duplicates
repet = []
for i in obj_path:
    if i.nums > 0:
        repet.append(l_path[obj_path.index(i)])
#len(repet) out:314
