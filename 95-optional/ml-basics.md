---
type: topic
title: Основы ML на уровне понятий
tier: 3
impact: 2
effort: 3
status: todo
confidence: 0
tags: [llm, tier3]
updated: 2026-10-29
---

# Основы ML на уровне понятий

> **Зачем на собесе:** Альфа и Constructor — «понимать ML»; AI-позиции могут спросить про эмбеддинги и метрики. Уровень — объяснить понятия, не обучать модели. Классический MLOps (MLflow, feature store, Kubeflow) — не нужен, если не целишься в ML-платформу.

## Минимум (стаб)

- Supervised/unsupervised, train/val/test, переобучение, кросс-валидация. Метрики: accuracy, precision/recall/F1, ROC-AUC; для ранжирования — recall@k, MRR, NDCG.
- Эмбеддинги: вектор смысла; cosine similarity; как получаются (трансформер); почему для русского языка модель надо проверять.
- LLM на пальцах: токены, контекстное окно, temperature/top-p, почему галлюцинирует, что такое fine-tuning vs RAG vs prompting, что даёт RLHF/instruction tuning, квантование моделей.
- Классический ML-пайплайн: данные → фичи → обучение → валидация → деплой → мониторинг дрейфа. Знать, что sklearn/pandas существуют и как выглядит inference-сервис (модель как артефакт, версия, FastAPI-обёртка, батчинг).
- Когда ML не нужен: правила/эвристики/SQL решают.

## Вопросы с собесов

- [ ] Что такое эмбеддинг и почему cosine? → *вектор, направление важнее длины*
- [ ] Precision vs recall — пример, когда что важнее? → *спам vs диагностика*
- [ ] Чем fine-tuning отличается от RAG, когда что? → *знания/стиль vs актуальные данные*
- [ ] Как бы задеплоил ML-модель как сервис? → *артефакт + версия, FastAPI, батчинг, мониторинг*

## Ресурсы

- Google ML Crash Course (быстро), «Hands-On ML» (Géron) — главы 1–4 по оглавлению
- Andrej Karpathy, «Intro to LLMs» (видео, 1 час)

## Связи

[rag](../70-llm-engineering/rag.md) · [evals-cost-latency](../70-llm-engineering/evals-cost-latency.md) · [vector-dbs](vector-dbs.md)

## Мои заметки

