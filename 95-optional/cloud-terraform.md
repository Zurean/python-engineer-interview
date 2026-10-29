---
type: topic
title: Облако и Terraform
tier: 3
impact: 2
effort: 4
status: todo
confidence: 0
tags: [infra, tier3]
updated: 2026-10-29
---

# Облако и Terraform

> **Зачем на собесе:** 1 из 7 (Constructor). Для РФ-рынка — отложить; K8s важнее. Для международной удалёнки — обязательно, и тогда выбрать одно облако (AWS по умолчанию) и пройти глубоко.

## Минимум для собеса (стаб)

- Одно облако глубоко: AWS — IAM (роли, политики, least privilege), VPC/подсети/security groups, EC2 vs ECS/EKS vs Lambda, RDS (Postgres, Multi-AZ, read replicas), S3 (presigned URL, lifecycle), SQS/SNS, CloudWatch, Secrets Manager. Аналоги в GCP/Yandex Cloud — уметь сопоставить.
- Terraform: providers, resources, state (remote backend + lock), modules, `plan`/`apply`, workspaces/окружения, drift. IaC в CI: `plan` в PR, `apply` после merge.
- Стоимость: понимать, что дорого (egress, NAT, over-provisioned RDS), теги и бюджеты.
- 12-factor, stateless, managed-сервисы вместо самодельных.

## Вопросы с собесов

- [ ] Как бы задеплоил FastAPI + Postgres + Redis в AWS? Что managed, что сам? → *ECS/EKS + RDS + ElastiCache + ALB; Terraform*
- [ ] Что такое Terraform state и почему он важен? → *источник правды, remote + lock*
- [ ] IAM: как дать сервису доступ к S3 без ключей в коде? → *роль на задачу/под (IRSA)*

## Ресурсы

- AWS Well-Architected Framework (обзорно), Terraform docs «Get Started»
- Yandex Cloud docs — если цель РФ с облаком

> Запросить углубление: «разверни `cloud-terraform` в хаб под AWS/Yandex Cloud».

## Связи

[[kubernetes]] · [[ci-cd]] · [[target-profile]]

## Мои заметки

