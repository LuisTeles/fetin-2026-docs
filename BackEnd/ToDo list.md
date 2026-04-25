
---
## Backend TODO List

### Módulo: Subjects (`/subjects`)
- [ ] `POST /subjects` — criar disciplina (RN-SUB-01, RN-SUB-02, RN-SUB-03)
- [ ] `GET /subjects` — listar disciplinas do usuário autenticado
- [ ] `GET /subjects/:id` — buscar disciplina por ID (validar ownership)
- [ ] `PATCH /subjects/:id` — atualizar nome / `priority_weight`
- [ ] `DELETE /subjects/:id` — excluir com guard: bloquear se existir schedule `active` (RN-SUB-05)
---

### Módulo: Topics (`/subjects/:subjectId/topics`)
- [ ] `POST /topics` — criar tópico; validar `parent_topic_id` pertence à mesma disciplina (RN-TOP-03); bloquear subtópico de subtópico (RN-TOP-04)
- [ ] `GET /topics` — listar tópicos de uma disciplina (com subtópicos aninhados)
- [ ] `GET /topics/:id` — buscar tópico por ID
- [ ] `PATCH /topics/:id` — atualizar; ao marcar `is_completed = true` no pai → marcar todos os filhos (RN-TOP-05)
- [ ] `DELETE /topics/:id` — excluir em cascata subtópicos (RN-TOP-06)

---

### Módulo: Exams (`/exams`)
- [ ] `POST /exams` — criar prova; rejeitar `exam_date` no passado (RN-EXM-01); unicidade `(subject_id, exam_date)` (RN-EXM-04)
- [ ] `GET /exams` — listar provas do usuário (com contagem dinâmica de tópicos pendentes — RN-EXT-03)
- [ ] `GET /exams/:id` — buscar prova com seus `exam_topics`
- [ ] `DELETE /exams/:id` — excluir + alterar schedules relacionados para `expired` (RN-EXM-05)
- [ ] `POST /exams/:id/topics` — vincular tópico à prova; validar mesma disciplina (RN-EXT-02); unicidade (RN-EXT-01)
- [ ] `DELETE /exams/:id/topics/:topicId` — desvincular tópico da prova

---

### Módulo: User Availability (`/availability`)
- [ ] `POST /availability` — criar bloco de disponibilidade; validar `end_time > start_time` (RN-AVL-02); duração mínima 30 min (RN-AVL-04); rejeitar sobreposição de horários (RN-AVL-03)
- [ ] `GET /availability` — listar disponibilidades do usuário
- [ ] `PATCH /availability/:id` — atualizar bloco (re-validar sobreposições)
- [ ] `DELETE /availability/:id` — excluir bloco

---

### Módulo: Schedules (`/schedules`) ← núcleo da aplicação
- [ ] `POST /schedules` — disparar geração do cronograma; executar o algoritmo em 7 etapas (Seção 4 do business-rules):
  1. Validação (tópicos vinculados, prazo ≥ 2 dias, sem schedule `active` para a mesma prova — RN-SCH-01, RN-SCH-05, RN-SCH-06)
  2. Leitura de `UserAvailability` → `available_minutes` por dia da semana
  3. Gerar registros em `ScheduleDay` para cada dia do período (RN-SCD-01, RN-SCD-02)
  4. Calcular `priority_score` por tópico e criar `ScheduleTopicAllocation` (RN-STA-01, RN-STA-02)
  5. Consultar `TopicRetention` para `spaced_review` sessions (RN-RET-05)
  6. Distribuir `StudySession` respeitando limites de minutos, prioridade e reserva dos últimos 1-2 dias para `pre_exam_review` (RN-STS-03, RN-STS-07)
  7. Persistir tudo em transação e definir `status = active`
- [ ] `GET /schedules` — listar cronogramas do usuário (com progresso calculado)
- [ ] `GET /schedules/:id` — buscar cronograma com dias e sessões
- [ ] `PATCH /schedules/:id/cancel` — cancelar cronograma (`active → cancelled` — RN-SCH-04)
- [ ] Cron job / trigger: transitar `active → completed` quando `exam_date` chegou (RN-SCH-04)

---

### Módulo: Schedule Days (`/schedules/:scheduleId/days`)
- [ ] `PATCH /schedule-days/:id` — marcar `is_available = false`; redistribuir sessões do dia para outros dias disponíveis (RN-SCD-03)

---

### Módulo: Study Sessions (`/study-sessions`)
- [ ] `PATCH /study-sessions/:id/complete` — marcar como `completed`; incrementar `sessions_completed` em `ScheduleTopicAllocation`; upsert `TopicRetention` com novo `next_review_at` (RN-STS-04, RN-STS-05, RN-RET-03, RN-RET-04)
- [ ] `PATCH /study-sessions/:id/skip` — marcar como `skipped`; não atualizar `TopicRetention`; reagendar em dia posterior (opcional — RN-STS-06)

---

### Módulo: Dashboard / Insights (read-only)
- [ ] `GET /dashboard` — dados consolidados: provas próximas, progresso por disciplina, retenção média, sessões pendentes do dia
- [ ] `GET /dashboard/retention` — estado de `TopicRetention` do usuário (todos os tópicos, scores, próximas revisões)

---

### Infraestrutura / Transversal
- [ ] Validação global via `class-validator` nos DTOs de todos os módulos
- [ ] Guards de ownership em todos os recursos (usuário só acessa seus próprios dados)
- [ ] Tratamento de erros padronizado (filtro global de exceções)
- [ ] Swagger/OpenAPI — documentar todos os endpoints (já existe swagger_auth.yaml para auth)
- [ ] Testes unitários dos services críticos (algoritmo de geração, cálculo de `priority_score`, progressão de `review_interval_days`)
- [ ] Testes e2e dos fluxos principais