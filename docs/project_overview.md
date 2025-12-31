## **Project Overview**
I'm developing a **bias-aware Early Warning System (EWS)** for higher education that predicts at-risk students while explicitly addressing algorithmic fairness—something most current systems fail to do adequately.

## **The Core Problem I'm Solving**
Current EWS have two critical flaws:
1. **Latency Problem**: They identify struggling students too late for effective intervention
2. **Bias Crisis**: They perpetuate systemic inequities, systematically disadvantaging students from underrepresented and economically disadvantaged communities (like the Wisconsin DEWS example where the system unfairly de-prioritized non-White students)

## **My Research Approach**

**Primary Research Question:**
How do socioeconomic, geographic, and demographic characteristics predict at-risk students, and what's the effectiveness of different bias mitigation approaches in reducing prediction disparities while maintaining strong predictive performance (AUC ROC > 0.80)?

**Three Key Objectives:**
1. **Audit** temporal EWS models for socioeconomic, geographic, and demographic bias
2. **Implement and compare** AI Fairness 360 (AIF360) mitigation techniques
3. **Evaluate** fairness-performance trade-offs while maintaining predictive power

## **My Methods**

**Dataset:** Open University Learning Analytics Dataset (OULAD) with 32,000+ students across seven modules

**Model:** Long Short-Term Memory (LSTM) network for temporal predictions

**Prediction Timing:** Within the first 25% of course completion (early enough to intervene effectively)

**Fairness Metrics:** ABROCA, Equalized Odds, Statistical Parity

**Performance Metric:** AUC ROC (targeting > 0.80)

## **Why This Matters**
Building fair, bias-aware early warning systems isn't just academically interesting—it's a regulatory and ethical imperative. With the U.S. Department of Education's 2024 guidance and the EU AI Act classifying educational AI as "high-risk," my work directly addresses the need to build systems that are both technically powerful AND ethically responsible.

My project represents a shift from treating fairness as an optional feature to making it a core requirement in educational AI systems.