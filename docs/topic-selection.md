# Topic and dataset selection

**Proposed title:** Personalized 60-Minute Glucose Forecasting Using CGMacros

**Research question:** How accurately can glucose be forecast 60 minutes ahead, and does calibration using a small amount of an individual's previous data improve forecasts compared with a shared population model? I will examine whether improvements differ between post-meal periods and other times.

**Dataset:** I will use [CGMacros](https://physionet.org/content/cgmacros/1.0.0/), a public dataset containing glucose measurements, meal timestamps and macronutrients, activity measurements, and participant information for 45 people with approximately ten days of observations each.

**Target variable:** The participant's Dexcom glucose value, measured in mg/dL, at the forecast timestamp.

**Forecast horizon:** 60 minutes ahead, using only information available at the prediction time.

**Data-access confirmation:** Downloaded and verified.
