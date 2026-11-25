# فهرست کامل فصل‌ها

۱۹ فایل · ۹۶ فصل

---

## [۰۱ — مقدمه و نقشهٔ آزمون](01-Introduction-and-Exam-Overview.md)

1. SecAI+ چیست و چه چیزی را می‌سنجد؟
2. مشخصات آزمون CY0-001 — قوانین بازی و منطق چهار دامنه
3. سه ستون SecAI+ — Securing AI، AI-Assisted Security، GRC و مرزهایشان
4. نقشهٔ راه پس از SecAI+ — از AI Red Teaming تا حکمرانی AI

## [۰۲ — مبانی AI برای امنیت](02-AI-Fundamentals-for-Security.md)

5. AI، ML، DL و GenAI — از دید یک امنیت‌کار
6. خط لولهٔ AI — داده، آموزش، استنتاج و نقاط تماس امنیتی
7. ترانسفورمر و مدل زبانی — آنچه باید بدانی، نه بیشتر
8. Embedding، Vector DB و RAG — و چرا از منظر امنیتی مهم‌اند
9. Prompt، System Prompt و پارامترها — سطح کنترل
10. Fine-tuning در برابر RAG در برابر Prompting — انتخاب و ریسک
11. کاربردهای AI در امنیت — نقشهٔ Use Case‌ها

## [۰۳ — چشم‌انداز تهدید AI](03-AI-Threat-Landscape.md)

12. مدل تهدید AI — دارایی، مهاجم، سطح حمله
13. OWASP Top 10 for LLM — نقشهٔ تهدیدها
14. MITRE ATLAS — زبان مشترک حمله به AI
15. طبقه‌بندی حملات — از داده تا استنتاج
16. تهدیدهای ناشی از AI — فیشینگ خودکار، بدافزار چندریختی، دیپ‌فیک
17. اقتصاد حمله و انگیزه‌ها — چه چیزی هدف است

## [۰۴ — Prompt Injection و Jailbreaking](04-Prompt-Injection-and-Jailbreaking.md)

18. Prompt Injection — چرا داده و دستور یکی هستند
19. Injection مستقیم — آناتومی و نمونه‌ها
20. Injection غیرمستقیم — سند، وب، ابزار، RAG
21. Jailbreak — خانواده‌ها و چرا کار می‌کنند
22. آزمون امنیتی Injection — روش Red Team
23. دفاع — جداسازی، سلسله‌مراتب دستور، فیلتر
24. Guardrails و Prompt Firewall — طراحی عملی

## [۰۵ — Poisoning داده و مدل](05-Data-and-Model-Poisoning.md)

25. Poisoning دادهٔ آموزش — چگونه و چقدر
26. Backdoor و Trigger — درِ پشتی مدل
27. Poisoning در RAG و پایگاه دانش
28. تشخیص و کاهش — از منبع تا مدل
29. Data Lineage و Provenance — زنجیرهٔ اعتماد داده

## [۰۶ — Model Inversion، Extraction و Theft](06-Model-Inversion-Extraction-Theft.md)

30. Model Inversion و Membership Inference
31. Model Extraction و سرقت مدل
32. حفظ داده در مدل — نشت از حافظه
33. دفاع — Differential Privacy، محدودسازی، اغتشاش خروجی
34. علامت‌گذاری و اثر انگشت مدل

## [۰۷ — امن‌سازی استقرار AI](07-Securing-AI-Deployment.md)

35. معماری استقرار — on-prem، cloud، hybrid و لایهٔ استنتاج
36. هویت و دسترسی برای AI — Least Privilege در عمل
37. کنترل شبکه و مرزها — egress، private endpoint، جداسازی
38. حفاظت از مدل، کلید و مصنوعات
39. لاگ، پایش و اتصال به SIEM
40. Rate Limit، Token Quota و کنترل هزینه

## [۰۸ — امنیت زنجیرهٔ تأمین AI](08-AI-Supply-Chain-Security.md)

41. زنجیرهٔ تأمین AI — اجزا و نقاط شکست
42. مدل‌های باز و مخزن‌ها — ارزیابی قبل از استفاده
43. امنیت خط لولهٔ MLOps و CI/CD
44. ریسک فروشنده و ارزیابی ثالث

## [۰۹ — AI در عملیات امنیتی](09-AI-in-Security-Operations.md)

45. اقتصاد SOC و جای AI
46. Triage خودکار هشدارها
47. همبستگی و کاهش هشدار تکراری
48. تشخیص ناهنجاری و UEBA
49. Human-in-the-loop در SOC

## [۱۰ — AI برای Threat Intelligence](10-AI-for-Threat-Intelligence.md)

50. غنی‌سازی و خلاصه‌سازی CTI
51. مهندسی تشخیص با AI
52. تحلیل لاگ و تولید کوئری
53. کمک به تحلیل بدافزار و رفتار
54. ریسک توهم در CTI — و راستی‌آزمایی

## [۱۱ — AI در آسیب‌پذیری و امنیت کد](11-AI-in-Vulnerability-and-Code-Security.md)

55. تحلیل و اولویت‌بندی آسیب‌پذیری
56. بازبینی کد با AI — قوت و مرز
57. تقویت SAST/DAST
58. کد تولیدشده با AI — ریسک‌های امنیتی
59. DevSecOps و دروازه‌های امنیتی

## [۱۲ — AI برای IR و SOAR](12-AI-for-IR-and-SOAR.md)

60. فازهای IR و نقش AI
61. Playbook خودکار و SOAR + AI
62. تولید گزارش و ارتباطات رخداد
63. Agentهای امنیتی — ریسک خودمختاری
64. سنجش اثربخشی — MTTD، MTTR، دقت

## [۱۳ — ریسک‌های AI-Assisted Security](13-Risks-of-AI-Assisted-Security.md)

65. سوگیری اتوماسیون و اعتماد بیش‌ازحد
66. تقویت منفی کاذب — وقتی AI تهدید را نمی‌بیند
67. نشت داده در ابزارهای امنیتی AI
68. دستکاری خصمانهٔ دفاع‌های AI

## [۱۴ — چارچوب‌های حاکمیت AI](14-AI-Governance-Frameworks.md)

69. حاکمیت، ریسک و انطباق — تفکیک
70. OECD AI Principles
71. NIST AI RMF — Govern، Map، Measure، Manage
72. ISO/IEC 42001، 23894 و رابطه با 27001
73. ساخت برنامهٔ حاکمیت AI

## [۱۵ — مقررات و حریم خصوصی](15-AI-Regulations-and-Privacy.md)

74. EU AI Act — سطوح ریسک و تعهدات
75. GDPR و حریم خصوصی در AI
76. آمریکا، بریتانیا و رویکردهای بخشی
77. مقررات سایر حوزه‌ها و اثر فرامرزی
78. استراتژی انطباق چندبازاری

## [۱۶ — مدیریت ریسک AI](16-AI-Risk-Management.md)

79. ارزیابی ریسک AI — روش‌ها
80. ثبت ریسک و راهبردهای درمان
81. ریسک ثالث و ارزیابی فروشنده
82. مستندسازی، ممیزی و رخداد AI

## [۱۷ — اخلاق، خط‌مشی و برنامه](17-AI-Ethics-Policy-and-Program.md)

83. اخلاق AI — از اصل تا تصمیم
84. نوشتن خط‌مشی AI سازمان
85. آموزش، فرهنگ و نقش‌ها
86. سنجش، ممیزی و بهبود مستمر

## [۱۸ — آزمایشگاه‌های یکپارچه](18-Integrated-Labs.md)

87. آزمایشگاه: تحلیل Prompt Injection و طراحی Guardrail
88. آزمایشگاه: Least Privilege برای یک AI API
89. آزمایشگاه: تشخیص Poisoning و Backdoor
90. آزمایشگاه: ارزیابی انطباق با NIST AI RMF
91. آزمایشگاه: Triage در SOC با AI و سنجش

## [۱۹ — آمادگی آزمون و واژه‌نامه](19-Exam-Prep-and-Glossary.md)

92. ساختار آزمون و استراتژی PBQ
93. تله‌های هر دامنه — خطاهای رایج
94. بیست سناریوی تمرینی با پاسخ تشریحی
95. برنامهٔ ۳۰ روزهٔ مرور
96. واژه‌نامه و چک‌لیست نهایی
