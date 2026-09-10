# 📊 Student Digital Life & Academic Performance Analysis

تحليل بيانات عن العلاقة بين أنماط استخدام الطلاب للتكنولوجيا (سوشيال ميديا، جيمينج، ستريمنج...) وعوامل حياتهم اليومية (نوم، رياضة، صحة نفسية...) وتأثير كل ده على **الأداء الأكاديمي (Final Exam Score)**، مع محاولة تعميم النتائج على داتا سيت تانية مختلفة تمامًا.

---

## 📁 محتويات المشروع

| الملف | الوصف |
|---|---|
| `predicting-digital-distraction-impact.ipynb` | نوتبوك التحليل الكامل (50 خلية) |
| `student_digital_life(First_data).csv` | الداتا سيت الأساسية (15,000 طالب) |
| `predict_student_performance(Second_data).csv` | داتا سيت ثانية (1,000 طالب) لاختبار تعميم النتائج |

---

## 🎯 هدف المشروع

1. استكشاف العلاقة بين ساعات الاستخدام الرقمي (موبايل، سوشيال ميديا، جيمينج...) والدرجة النهائية.
2. فهم تأثير عوامل زي الصحة النفسية، التعليم الوالدي، جودة الإنترنت على الأداء الدراسي.
3. بناء "معيار تنبؤ" (lookup table) مبني على متوسط الدرجات حسب عدد ساعات المذاكرة، واختباره على داتا سيت تانية مختلفة الأعمدة والتوزيع.

---

## 🗂️ وصف الداتا سيت الأساسية (`student_digital_life.csv`)

- **الحجم:** 15,000 صف × 18 عمود، بدون قيم مفقودة (Non-Null بالكامل).

| العمود | الوصف |
|---|---|
| `student_id`, `age`, `gender` | بيانات ديموغرافية |
| `study_hours_per_day` | ساعات المذاكرة يوميًا |
| `smartphone_usage_hours`, `social_media_hours`, `gaming_hours`, `streaming_hours` | ساعات الاستخدام الرقمي |
| `sleep_hours`, `exercise_hours`, `caffeine_intake_cups` | نمط الحياة |
| `class_attendance_percent`, `assignment_completion_percent` | الانتظام الدراسي |
| `mental_health_status`, `internet_quality` | (Poor / Average / Good) |
| `parent_education_level` | (HighSchool / Bachelors / Masters / PhD) |
| `motivation_level` | مستوى الدافعية |
| `final_exam_score` | **المتغير الهدف** |

---

## 🔄 خطوات التحليل (Workflow)

### 1. الاستكشاف الأولي (EDA)
- `df.info()` و `df.describe()` لفهم أنواع الأعمدة وتوزيعها.
- Boxplots لكل الأعمدة الرقمية لاكتشاف الـ Outliers.

### 2. معالجة القيم الشاذة (Outlier Handling)
- استخدام طريقة **IQR** (Interquartile Range) على 10 أعمدة رئيسية، مع عمل **Clipping** للقيم اللي برا الحدود (upper/lower bound) بدل حذفها.
- إعادة رسم الـ Boxplots للتأكد من نجاح المعالجة.

### 3. تحليل الارتباط (Correlation Analysis)
- Heatmap شامل لكل الأعمدة الرقمية.
- Heatmap مركّز على علاقة كل عمود بـ `final_exam_score` تحديدًا.
- تحويل `mental_health_status` لقيم رقمية (Poor=0, Average=1, Good=2) لقياس ارتباطها بالدرجة.

### 4. تصور العلاقات (Visualization)
- خطوط بيانية (Line plots) لمتوسط الدرجة مقابل كل متغير رقمي (بعد تجميع القيم في فئات صغيرة بفاصل 0.5).
- Bar chart تفاعلي (Plotly) لمتوسط الدرجة حسب الحالة النفسية.
- Scatter plot مع خط انحدار (OLS trendline) لعلاقة ساعات المذاكرة بالدرجة النهائية.

### 5. تنظيف الداتا سيت الثانية (`predict_student_performance.csv`)
- **الحجم الأصلي:** 1,000 صف × 12 عمود، بها قيم مفقودة في كل الأعمدة تقريبًا.
- ملء `StudentID` المفقود بأرقام تسلسلية جديدة.
- ملء `Gender` و `Name` المفقودين بقيمة `"Unknown"`.
- حذف عمودين مكرّرين (`Study Hours`, `Attendance (%)`) لوجود بدائل لهم.
- ملء الأعمدة الرقمية (`AttendanceRate`, `StudyHoursPerWeek`, `PreviousGrade`, `ExtracurricularActivities`, `FinalGrade`) بالمتوسط (Mean).
- ملء الأعمدة الفئوية (`ParentalSupport`, `Online Classes Taken`) بالـ Mode.

### 6. بناء "معيار التنبؤ" والتحقق منه (Cross-Dataset Validation)
- تحويل `StudyHoursPerWeek` للداتا سيت الثانية إلى `study_hours_per_day` (بالقسمة على 7) عشان يتوافق مع الداتا سيت الأولى.
- تقسيم ساعات المذاكرة لفئات (bins) بفاصل ساعة واحدة (0 → 10).
- حساب متوسط `final_exam_score` لكل فئة في الداتا الأولى، واستخدامه كـ **Lookup Table** للتنبؤ بدرجة الداتا الثانية بناءً على نفس الفئة.
- تكرار نفس الفكرة بالعكس: استخدام متوسط `FinalGrade` من الداتا الثانية للتنبؤ بدرجة تقديرية للداتا الأولى، مع معالجة الحالات الطرفية (مثلاً: 0 ساعات مذاكرة = "Very bad grade").

---

## 🛠️ الأدوات المستخدمة

- **Python**: `pandas`, `numpy`
- **التصور البياني**: `matplotlib`, `seaborn`, `plotly.express`
- **البيئة**: Jupyter Notebook

---

## ⚠️ ملاحظات ونقاط تحتاج تحسين

- **مفيش ML حقيقي هنا** — أسلوب "التنبؤ" المستخدم هو Lookup Table مبني على متوسط الدرجة داخل كل فئة ساعات مذاكرة، مش Machine Learning model. لو الهدف فعلاً "Predict"، الخطوة الطبيعية بعد كده هي تدريب موديل انحدار (Linear Regression / Random Forest) على الفيتشرز المشتركة بين الداتا سيتين.
- في تكرار لنفس كود الـ Boxplot والـ Heatmap في أكتر من خلية (Cell 4/6 و Cell 7/8) — ممكن تتحول لدالة واحدة (function) تتنادى براحتك.
- الـ paths بتاعة قراءة الـ CSV مكتوبة Hardcoded (`C:\Users\Moham\...`) — أفضل تتحول لمسار نسبي (relative path) عشان النوتبوك يشتغل على أي جهاز.
- خلية 10 (custom_orders) فيها كود مقطوع (الـ for loop مش مكتمل في الكود اللي وصلني) — يستاهل مراجعة.
- الداتا الثانية فيها Missing values بنسبة كبيرة (حوالي 4% في كل عمود)، فمليها بالـ Mean/Mode حل سريع لكنه ممكن يقلل من التباين الحقيقي في البيانات.

---

## ▶️ طريقة التشغيل

```bash
pip install pandas numpy matplotlib seaborn plotly
jupyter notebook Project_2.ipynb
```

تأكد من تعديل مسارات ملفات الـ CSV في أول خليتين ليطابقوا مكان الملفات عندك.
