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













