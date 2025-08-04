# Walmart_Store_Sales_Forecasting
## Problem Definition
მონაცემებში წარმოდგენილია Walmart-ის გაყიდვები 45 სხვადასხვა მაღაზიაში, რომლებიც სხვადასხვა რეგიონებში მდებარეობს. თითოეულ მაღაზიას აქვს რამდენიმე დეპარტამენტი და დავალებაა დეპარტამენტის ყოველკვირეული გაყიდვების პროგნოზი თითოეულ მათგანში.

Walmart მთელი წლის განმავლობაში აკეთებს სხვადასხვა ტიპის ფასდაკლებებს. ეს ფასდაკლებები წინ უსწრებს მნიშვნელოვან დღესასწაულებს: Super Bowl, Labor Day,Thanksgiving, ახალი წელი და შობა.

- train.csv - სატრენინგო მონაცემები (მაღაზია, დეპარტამენტი, თარიღი, გაყიდვები)
- test.csv - სატესტო მონაცემები პროგნოზისთვის
- features.csv - დამატებითი ფაქტორები (ტემპერატურა, საწვავის ფასი, CPI, უმუშევრობა, ფასდაკლებები)
- stores.csv - მაღაზიების მეტამონაცემები (ტიპი, ზომა)

კვლევის ძირითადი მიზანი იყო სხვადასხვა მოდელის ერთმანეთისთვის შედარება და მათი ანალიზი. ამიტომაც, შევარჩიე შემდეგი მოდელები:
- SARIMA
- DLinear
- XGBoost
- Linear Regression
- Random Forest
  
ჩემი  მიდგომა მოიცავდა შემდეგ ნაბიჯებს:

- მონაცემთა წინასწარი დამუშავება: მონაცემთა გაწმენდა, NaN-ების ჩანაცვლება/შევსება და კატეგორიული ცვლადების გარდაქმნა.
- feature-ების ინჟინერია: ახალი feature-ების შექმნა, რომლებიც ასახავს დროის ტენდენციებს, სეზონურობას და სხვა ფაქტორებს.
- მოდელების ტრენინგი: მოდელების დატრენინგება train data-ზე და მათი მეტრიკების შეფასება.
- ჰიპერპარამეტრების ოპტიმიზაცია: მოდელების მუშაობის გაუმჯობესება ჰიპერპარამეტრების ოპტიმიზაციის სხვადასხვა მეთოდის გამოყენებით.
- MLflow tracking: ექსპერიმენტების დალოგვა

## რეპოზიტორიის სტრუქტურა
1. model_experiment_DLinear.ipynb
2. model_experiment_SARIMA.ipynb
3. model_experiment_XGBoost.ipynb
4. model_inference_xgboost.ipynb
5. README.md
   
.ipynb ფაილები შეიცავს Notebook-ებს თითოეული მოდელისთვის, სადაც დეტალურადაა აღწერილი მონაცემთა დამუშავება, feature engineering/selection, მოდელის ტრენინგი და მათი მეტრიკები.

readme.md - პროექტის მიმოხილვა

## Feature Engineering
### კატეგორიული ცვლადების რიცხვითში გადაყვანა
- მაღაზიის ტიპი (Type): გარდავქმენი კატეგორიულ ცვლადად (category) და გამოვიყენე One-Hot Encoding (A, B, C ტიპებისთვის),ეს საშუალებას აძლევს მოდელს, გაითვალისწინოს მაღაზიის ტიპის გავლენა მის გაყიდვებზე.
- დღესასწაულის მაჩვენებელი (IsHoliday): გარდავქმენი ბინარულ ცვლადად (0/1), სადაც 1 მიუთითებს დღესასწაულზე და 0 ჩვეულებრივ დღეზე... გაერთიანების შედეგად წარმოქმნილი ორი IsHoliday სვეტი (IsHoliday_x, IsHoliday_y) გავაერთიანე ერთ სვეტად, ამოვიღე ზედმეტი სვეტები და გარდავქმენი int ტიპად.

### NAN მნიშვნელობების დამუშავება
- MarkDown სვეტები: MarkDown1-დან MarkDown5-მდე სვეტებში NaN მნიშვნელობები შევავსე 0-ით, რადგან ეს სვეტები წარმოადგენს ფასდაკლებებს, რომლებიც, თუ NaN-ია, ჩავთვალე 0-ად.
- CPI და უმუშევრობა (Unemployment): NaN მნიშვნელობები შევავსე თითოეული სვეტის მედიანური მნიშვნელობით, რათა თავიდან ამეცილებინა გადახრები მონაცემებში.

### Cleaning მიდგომები
- ამოვიღე ჩანაწერები, სადაც Weekly_Sales უარყოფითი იყო (შესაძლოა რამე შეცდომაა ან თანხის დაბრუნებას ნიშნავს, რაც არ გამომადგებოდა მოდელის დატრენინგებაში)
- Handling outliers: გამოვიყენე IQR (Interquartile Range) მეთოდი გაყიდვების outlier-ების 'დასაჭერად'.
- outlier-ები, რომლებიც არ იყო დღესასწაულის/ნოემბერ-დეკემბრის პერიოდში, შევცვალე ზედა ზღვრით, რათა თავიდან ამეცილებინა მოდელის overfitting.

## Feature Selection
- train_full და test_full დავალაგეთ Store, Dept და Date-ის მიხედვით, რათა სწორად გამოვთვალო ლაგ-ფიჩერები.
- გამოვიყენე კორელაციის ანალიზი, რათა გამოვავლინოთ ყველაზე მნიშვნელოვანი ფიჩერები.

<img width="1040" height="692" alt="image" src="https://github.com/user-attachments/assets/25f05a3f-363e-4a9c-a6d8-63276bc5888d" />

<img width="1039" height="462" alt="image" src="https://github.com/user-attachments/assets/af54590e-561c-4ee1-9769-42abaeaa9195" />

<img width="869" height="764" alt="image" src="https://github.com/user-attachments/assets/b65520a5-83c6-4c8a-8c60-0e69e8a90c8f" />

## Training
### ტესტირებული მოდელები:

დავატრენინგე ხუთი განსხვავებული მოდელი:
1. Linear Regression: საბაზისო მოდელი, რომელიც კარგად იჭერს წრფივ დამოკიდებულებებს. (ამ პრობლემისთვის არც თუ ისე შესაფერისი...)
2. Random Forest: მოდელი, რომელიც ეფექტურია არაწრფივი დამოკიდებულების დასაფიქსირებლად.
3. XGBoost: წინა ორ მოდელთან შედარებით გამოირჩევა მაღალი სიზუსტით
4. SARIMA: time series მოდელი, რომელიც ითვალისწინებს სეზონურობას
5. DLinear: წრფივი ნეირონული მოდელი დროის მწკრივებისთვის

### Linear Regression:

Linear Regression გამოვიყენე საბაზისო მოდელად, რომელიც ცდილობს დაიჭიროს წრფივი დამოკიდებულება Weekly_Sales-სა და სხვადასხვა featureb-ს შორის (მაგ., Store_Sales_Mean, HolidaySeason, CPI), ამის გამო ამ მოდელის დატრენინგება არ იყო კარგი იდეა ამ პრობლემისთვის...

შედეგები:
- Linear Regression (Train) Metrics:
  - MAE: 7160.7770
  - MSE: 136859787.0574
  - RMSE: 11698.7088
  - R2: 0.6523
  - MAPE: 12529.0283
  - WMAE: 7160.7770
- Linear Regression (Val) Metrics:
  - MAE: 6735.7998
  - MSE: 89510679.6375
  - RMSE: 9461.0084
  - R2: 0.7218
  - MAPE: 36485.8000
  - WMAE: 6735.7998
 
https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/1/runs/f1a17c382ebe417897e4752d783284a6

https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/1?searchFilter=&orderByKey=attributes.start_time&orderByAsc=false&startTime=ALL&lifecycleFilter=Active&modelVersionFilter=All+Runs&datasetsFilter=W10%3D

<img width="986" height="608" alt="image" src="https://github.com/user-attachments/assets/68864426-9a6f-42ed-9ffd-2c90eac314b6" />

   
### Random Forest

Random Forest აერთიანებს 100 გადაწყვეტილების ხეს არაწრფივი ურთიერთობების დასაფიქსირებლად. გამოვიყენე პარამეტრები: n_estimators=100, max_depth=15, min_samples_split=5, min_samples_leaf=2, random_state=42 და n_jobs=-1 .
პარამეტრების საუკეთესო კომბინაცია იყო n_estimators=100, max_depth=15 და min_samples_split=5, რამაც გააუმჯობესა მოდელის სიზუსტე და შეამცირა overfitting

შედეგები: 
- Random Forest (Train) Metrics:
  - MAE: 1230.6036
  - MSE: 6340282.2819
  - RMSE: 2517.9917
  - R2: 0.9839
  - MAPE: 1079.2664
  - WMAE: 1230.6036
- Random Forest (Val) Metrics:
  - MAE: 1661.9067
  - MSE: 9792586.2384
  - RMSE: 3129.3108
  - R2: 0.9696
  - MAPE: 3713.2180
  - WMAE: 1661.9067
 
https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/2/runs/d929068ee28b462b844e078c7789c66d

https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/2?searchFilter=&orderByKey=attributes.start_time&orderByAsc=false&startTime=ALL&lifecycleFilter=Active&modelVersionFilter=All+Runs&datasetsFilter=W10%3D

<img width="789" height="492" alt="image" src="https://github.com/user-attachments/assets/f0145e85-d281-4c55-bc3b-c5ccc59ee002" />

### XGBoost
მოდელი დავატრენინგე შემდეგი ჰიპერპარამეტრებით:
xgb_params = {
        'n_estimators': 500,
        'max_depth': 6,
        'learning_rate': 0.1,
        'subsample': 0.8,
        'colsample_bytree': 0.8,
        'random_state': 42,
        'n_jobs': -1,
        'early_stopping_rounds': 50,
        'eval_metric': 'mae'
    }
მოდელი ტრენინგდა იმავე 80%-იან dataზე,overfitting-ის თავიდან ასაცილებლად

შედეგები: 
- XGBoost (Train) Metrics:
  - MAE: 1859.0164
  - MSE: 9748211.0595
  - RMSE: 3122.2125
  - R2: 0.9752
  - MAPE: 2419.1423
  - WMAE: 1859.0164
- XGBoost (Val) Metrics:
  - MAE: 2290.7544
  - MSE: 14200514.7450
  - RMSE: 3768.3570
  - R2: 0.9559
  - MAPE: 7993.1636
  - WMAE: 2290.7544
 
  https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/3/runs/fd3eda7176954ba881ba2f2f41b2babf

  https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/3

  <img width="787" height="491" alt="image" src="https://github.com/user-attachments/assets/d4811982-b433-4f73-bd82-6ff79e10e93e" />


მოდელების შესადარებლად გამოვიყენოთ შემდეგი გრაფიკები:

<img width="1188" height="592" alt="image" src="https://github.com/user-attachments/assets/a0988d95-5a8b-4eed-98a4-b3d6d65c0598" />

<img width="1395" height="595" alt="image" src="https://github.com/user-attachments/assets/e2499faa-fa58-4c97-bdba-1253fd6da327" />

<img width="1062" height="73" alt="image" src="https://github.com/user-attachments/assets/58dd2c67-7e64-4ddc-ba4d-b9ab6e63cc86" />

<img width="1040" height="72" alt="image" src="https://github.com/user-attachments/assets/6cc7bd3a-43c6-4551-9959-dbf633d6e1fe" />


### SARIMA
გამოვიყენეთ SARIMA მოდელი Walmart-ის გაყიდვების (Weekly Sales) პროგნოზირებისთვის, როგორც time series ანალიზის მეთოდი, რომელიც ითვალისწინებს სეზონურობას. 

SARIMAX მოდელი გამოიყen ARIMA-ს 'გაუმჯობესებული' ვერსია, რომელიც ითვალისწინებს სეზონურ კომპონენტებს და გარე ცვლადებს, როგორიცაა Temperature, CPI, Unemployment და MarkDown-ები. მოდელის პარამეტრები (p, d, q) და სეზონური პარამეტრები (P, D, Q, m) ავტომატურად განისაზღვრა pmdarima.auto_arima-ს მეშვეობით, სადაც ( m = 52 ) (წლიური სეზონურობა, რადგან მონაცემები კვირეულია).

პარამეტრების ავტომატური შერჩევა-  ყოველი მაღაზიისთვის გამოვიყენეთ auto_arima, რათა მიეღწა ოპტიმალური (p, d, q) და (P, D, Q, m) პარამეტრები. ( max_p = 2 ), ( max_q = 2 ), ( max_P = 1 ), ( max_Q = 1 ) და ( m = 52 ) განისაზღვრა სეზონური პერიოდის საფუძველზე. 

<img width="979" height="447" alt="image" src="https://github.com/user-attachments/assets/f2eaa4c6-dba3-464a-a9a6-bbd94ad29fa4" />


### DLinear
DLinear  არის deep learning-ის ერთ-ერთი მოდელი,რომელიც დაფუძნებულია time series ანალიზზე. მოდელი ითვალისწინებს სეზონურ და 'ტრენდულ' (ტენდენციური კომპონენტები) კომპონენტებს, ასევე გარე ფაქტორებს. ყველა ექსპერიმენტი დაფიქსირდა MLflow-ით Dagshub-ის რეპოზიტორიაში

https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/0/runs/7b23c5e4c9eb43c08c724cf4f448cc9e

https://dagshub.com/jgushiann/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments/0

პარამეტრები:
- SEQ_LEN = 4
- PRED_LEN = 2
- BATCH_SIZE = 64
- EPOCHS = 50
- LEARNING_RATE = 0.001
- PATIENCE = 10
- TEST_SIZE = 0.1
- VAL_SIZE = 0.05

თავიდან, train.csv, stores.csv და features.csv გააერთიანდა train_full-ში, ხოლო test.csv - test_full-ში, left join-ის გამოყenებით.
მერჯის შემდეგ წარმოიქმნა ორი იდენტური სვეტი IsHoliday_x და IsHoliday_y, რომელიც გავაერთიანე ერთ ცალკე სვეტად(IsHoliday).
განსხვავებით linar/random forest/xgboost მოდელებისგან, აქ არ წაგვიშლია უარყოფითი Weekly_Sales-ის მნიშვნელობები, თუმცა გავფილტრეთ IQR მეთოდით.

შედეგები:
- MAE: 2430.1650
  - RMSE: 4001.7834
  - MAPE: 10758.0645
  - R2: 0.9666

 <img width="1725" height="522" alt="image" src="https://github.com/user-attachments/assets/a664f6a1-407f-4866-841a-044b04584968" />

 <img width="1719" height="387" alt="image" src="https://github.com/user-attachments/assets/8ef0ef03-d09f-402c-8ea8-48e88003ac11" />













