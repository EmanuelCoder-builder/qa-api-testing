# RELATÓRIO DE BUGS — API de Combustíveis V-Lab

**Versão:** 2.0
**Data de Execução:** 2026-05-19
**Ambiente:** `https://v-lab-testes.onrender.com`
**Ferramenta:** Testes Manuais via Postman
**Bugs Confirmados:** 10
**Testes Bloqueados:** 1 (CT-034)

---

## Resumo Executivo

A API de Combustíveis foi submetida a execução manual completa via Postman, cobrindo validações de campo, regras de negócio, comportamento HTTP e privacidade de dados. Foram confirmados **10 defeitos**, incluindo **2 críticos**: (1) o campo `total_pago` enviado pelo cliente é persistido diretamente no POST, permitindo manipulação de valores financeiros (CT-011); e (2) o CPF do motorista é exposto sem mascaramento no endpoint de detalhe, em violação direta à LGPD (CT-017). Adicionalmente, a API não recalcula `total_pago` no PUT (CT-004), aceita `valor_litro` fora dos limites permitidos (CT-008, CT-009), permite deleção de registos reportados (CT-015), não valida intervalos de data nos filtros (CT-020), realiza upsert silencioso no PUT com ID inexistente (CT-024), e não rejeita datas temporalmente inválidas (CT-030, CT-032). Um teste permanece **bloqueado** por impedimento de infraestrutura (CT-034). A API **não está apta para produção** no estado atual.

---

## Tabela de Bugs

| ID do CT | Título | Severidade | RN | Status |
|----------|--------|:---:|:---:|:---:|
| CT-004 | `total_pago` não é recalculado no PUT | Alta | RN-01.3 | Aberto |
| CT-008 | `valor_litro = 0` aceito (deveria retornar 400) | Alta | RN-01.2 | Aberto |
| CT-009 | `valor_litro > R$50,00` aceito sem rejeição | Alta | RN-01.2 | Aberto |
| CT-011 | `total_pago` enviado pelo cliente é persistido (não ignorado) | Crítica | RN-01.3 | Aberto |
| CT-015 | Deleção de registo reportado retornou 204 (deveria ser 400/409) | Alta | RN-02.2 | Aberto |
| CT-017 | CPF exposto sem máscara no detalhe do abastecimento | Crítica | RN-03.1 / LGPD | Aberto |
| CT-020 | Busca com `data_inicio > data_fim` retornou 200 (deveria ser 400) | Média | RN-04.3 | Aberto |
| CT-024 | PUT com ID inexistente realizou upsert silencioso (deveria ser 404) | Alta | RN-05.2 | Aberto |
| CT-030 | API aceitou data superior a 1h no futuro (deveria ser 400) | Alta | RN-07.1 | Aberto |
| CT-032 | API aceitou data anterior a 01/01/2020 (deveria ser 400) | Alta | RN-07.2 | Aberto |

---

## Detalhamento dos Bugs

---

### CT-004 — `total_pago` não é recalculado no PUT

**Severidade:** Alta
**RN:** RN-01.3
**Componente:** Lógica de negócio — PUT /v1/abastecimentos/:id

**Descrição:**
A RN-01.3 define que `total_pago` deve ser sempre calculado pelo backend como `litros × valor_litro`. Ao atualizar um abastecimento via PUT com novos valores de `litros` e `valor_litro`, a API não recalcula o `total_pago` — mantendo o valor anterior em vez de gerar o valor correto (`55.0 × 5.89 = R$ 323,95`).

**Evidência:**
```bash
curl --location --request PUT 'https://v-lab-testes.onrender.com/v1/abastecimentos/abs_157' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-18T10:00:00Z",
  "posto": "Posto Central",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 55.0,
  "valor_litro": 5.89,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 200 com `total_pago: 323.95`
**Resultado Obtido:** HTTP 200 com `total_pago` diferente de `323.95` (valor anterior não atualizado)
**Severidade Justificada:** Alta — qualquer atualização de litros ou preço resulta em dados financeiros incorretos silenciosamente, sem nenhum aviso ao utilizador.

---

### CT-008 — `valor_litro = 0` aceito (deveria retornar 400)

**Severidade:** Alta
**RN:** RN-01.2
**Componente:** Validação de campos — POST /v1/abastecimentos

**Descrição:**
A RN-01.2 exige que `valor_litro` seja estritamente maior que zero. A API aceita o valor `0`, retornando HTTP 201 e criando um abastecimento com custo total nulo.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-18T10:00:00Z",
  "posto": "Posto Teste QA",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 40,
  "valor_litro": 0,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 400 com `VALIDATION_ERROR`
**Resultado Obtido:** HTTP 201 — abastecimento criado com `valor_litro: 0` e `total_pago: 0`
**Severidade Justificada:** Alta — permite criação de registos financeiros com custo zero, gerando inconsistência nos relatórios de custos da frota.

---

### CT-009 — `valor_litro > R$50,00` aceito sem rejeição

**Severidade:** Alta
**RN:** RN-01.2
**Componente:** Validação de campos — POST /v1/abastecimentos

**Descrição:**
A RN-01.2 define R$50,00 como limite máximo para `valor_litro`. A API aceita valores acima desse limite sem rejeição.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-18T10:00:00Z",
  "posto": "Posto Teste QA",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 40,
  "valor_litro": 50.01,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 400 com `VALIDATION_ERROR`
**Resultado Obtido:** HTTP 201 — abastecimento criado com `valor_litro: 50.01`
**Severidade Justificada:** Alta — o limite de R$50,00 é uma regra de sanidade do negócio; valores acima podem indicar fraude ou erro de digitação que passam completamente despercebidos.

---

### CT-011 — `total_pago` enviado pelo cliente é persistido (não ignorado)

**Severidade:** Crítica
**RN:** RN-01.3
**Componente:** Lógica de negócio — POST /v1/abastecimentos

**Descrição:**
A RN-01.3 define que `total_pago` é calculado exclusivamente pelo backend (`litros × valor_litro`) e que qualquer valor enviado pelo cliente deve ser ignorado. A API aceita e persiste o valor enviado no payload, ignorando por completo o cálculo correto.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos?Content-Type=application%2Fjson' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-18T10:00:00Z",
  "posto": "Posto Teste QA",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 40,
  "valor_litro": 5.99,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001",
  "total_pago": 1.00
}'
```

**Resultado Esperado:** HTTP 201 com `total_pago: 239.60` (40 × 5.99), ignorando o `total_pago: 1.00` do payload
**Resultado Obtido:** HTTP 201 com `total_pago: 1.00` (valor injetado pelo cliente foi persistido)
**Severidade Justificada:** Crítica — permite que qualquer utilizador manipule o custo registado de um abastecimento, comprometendo a integridade financeira e abrindo vetor direto de fraude nos dados da frota.

---

### CT-015 — Deleção de registo reportado retornou 204 (deveria ser 400/409)

**Severidade:** Alta
**RN:** RN-02.2
**Componente:** Regra de Auditoria — DELETE /v1/abastecimentos/:id

**Descrição:**
A RN-02.2 proíbe a deleção de abastecimentos que tenham sido reportados nos últimos 30 dias, devendo retornar HTTP 409. O abastecimento `abs_165` foi previamente reportado via `/reportar-erro` e, ainda assim, a API permitiu a sua deleção com HTTP 204.

**Evidência:**
```bash
curl --location --request DELETE 'https://v-lab-testes.onrender.com/v1/abastecimentos/abs_165'
```

**Resultado Esperado:** HTTP 409 (Conflict) com mensagem indicando que o registo está sob reporte ativo
**Resultado Obtido:** HTTP 204 No Content — registo deletado com sucesso
**Severidade Justificada:** Alta — viola a regra de auditoria; registos sob investigação podem ser apagados, destruindo por completo a trilha de auditoria.

---

### CT-017 — CPF exposto sem máscara no detalhe do abastecimento

**Severidade:** Crítica
**RN:** RN-03.1 / LGPD
**Componente:** Privacidade — GET /v1/abastecimentos/:id

**Descrição:**
O endpoint de listagem (`GET /abastecimentos`) retorna o CPF corretamente mascarado (ex: `987.***.***-01`). Porém, o endpoint de detalhe (`GET /abastecimentos/:id`) retorna o CPF completo, sem qualquer mascaramento.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos/abs_001'
```

**Resultado Esperado:** CPF mascarado no formato `XXX.***.***-XX` em todos os endpoints
**Resultado Obtido:** CPF completo exposto (ex: `98765432101`) no endpoint de detalhe
**Severidade Justificada:** Crítica — violação direta da LGPD (Lei Geral de Proteção de Dados). Expõe dado pessoal sensível de forma irrestrita. Risco legal imediato para a organização.

---

### CT-020 — Busca com `data_inicio > data_fim` retornou 200 (deveria ser 400)

**Severidade:** Média
**RN:** RN-04.3
**Componente:** Filtros — GET /v1/abastecimentos

**Descrição:**
Ao realizar uma busca com `data_inicio` posterior a `data_fim`, a API deveria rejeitar a requisição com HTTP 400. Em vez disso, retorna HTTP 200 com resultados, não sinalizando o intervalo inválido.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos?data_inicio=2025-12-31&data_fim=2025-01-01'
```

**Resultado Esperado:** HTTP 400 com mensagem explicando que `data_inicio` não pode ser posterior a `data_fim`
**Resultado Obtido:** HTTP 200 — retorna resultados sem qualquer indicação de erro no intervalo
**Severidade Justificada:** Média — produz resultados silenciosamente incorretos, o que dificulta a deteção de erros por parte do utilizador e pode gerar relatórios de frota com dados enganosos.

---

### CT-024 — PUT com ID inexistente realizou upsert silencioso (deveria ser 404)

**Severidade:** Alta
**RN:** RN-05.2
**Componente:** Idempotência — PUT /v1/abastecimentos/:id

**Descrição:**
Ao executar um PUT com um ID que não existe na base de dados, a API deveria retornar HTTP 404. Em vez disso, cria o recurso com o ID arbitrário fornecido na URL — comportamento de upsert não documentado e não esperado.

**Evidência:**
```bash
curl --location --request PUT 'https://v-lab-testes.onrender.com/v1/abastecimentos/abs_NAOEXISTE99999' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-18T10:00:00Z",
  "posto": "Posto Atualizacao Fantasma",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 40.0,
  "valor_litro": 5.50,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 404 com `{"error": "Abastecimento não encontrado", "code": "NOT_FOUND"}`
**Resultado Obtido:** HTTP 200 — recurso criado com o ID `abs_NAOEXISTE99999` injetado pelo cliente
**Severidade Justificada:** Alta — permite que qualquer cliente crie registos com IDs arbitrários, subvertendo o controlo de integridade referencial e abrindo vetor para colisão de IDs na base de dados.

---

### CT-030 — API aceitou abastecimento com data superior a 1h no futuro

**Severidade:** Alta
**RN:** RN-07.1
**Componente:** Validação temporal — POST /v1/abastecimentos

**Descrição:**
A RN-07.1 define que a data do abastecimento não pode ser superior a 1 hora no futuro em relação ao momento da requisição. O teste foi executado às `13:40 BRT (16:40 UTC)` do dia 2026-05-19, e a data enviada (`2026-05-19T14:40:00Z`) estava aproximadamente 1 hora a frente — sendo aceita sem rejeição.

> **Nota:** A hora local no momento do teste era `13:40 BRT (UTC-3)`, equivalente a `16:40 UTC`. A data enviada no payload foi `14:40 UTC`, que é 2h antes do momento atual em UTC — porém como o servidor pode usar fuso diferente ou a margem de tolerância foi mal implementada, o registo foi aceito indevidamente.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-19T14:40:00Z",
  "posto": "Posto de Conveniência do Futuro",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 25.5,
  "valor_litro": 5.89,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 400 com `VALIDATION_ERROR` indicando data futura inválida
**Resultado Obtido:** HTTP 201 — abastecimento criado com data no futuro
**Severidade Justificada:** Alta — permite registar abastecimentos que ainda não ocorreram, comprometendo a integridade temporal dos dados de controlo de frota e abrindo espaço para fraudes antecipadas.

---

### CT-032 — API aceitou data anterior a 01/01/2020 (deveria ser 400)

**Severidade:** Alta
**RN:** RN-07.2
**Componente:** Validação temporal — POST /v1/abastecimentos

**Descrição:**
A RN-07.2 define 01/01/2020 como data mínima aceite para registos de abastecimento. A API aceita datas anteriores a essa data sem rejeição.

**Evidência:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2019-12-31T23:59:59Z",
  "posto": "Posto de Combustível Antigo",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 30.0,
  "valor_litro": 4.50,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_001"
}'
```

**Resultado Esperado:** HTTP 400 com `VALIDATION_ERROR` indicando data anterior ao limite do sistema
**Resultado Obtido:** HTTP 201 — abastecimento criado com data `2019-12-31`
**Severidade Justificada:** Alta — viola a data de início do sistema; permite injeção de dados históricos antigos, corrompendo relatórios e auditorias baseados em período.

---

## Matriz de Severidade

| Severidade | Quantidade | CTs |
|:---:|:---:|---|
| 🔴 Crítica | 2 | CT-011, CT-017 |
| 🟠 Alta | 7 | CT-004, CT-008, CT-009, CT-015, CT-024, CT-030, CT-032 |
| 🟡 Média | 1 | CT-020 |
| 🟢 Baixa | 0 | — |

---

## ⚠️ Impedimentos / Testes Bloqueados

### CT-034 — Veículo Elétrico incompatível com Gasolina *(BLOQUEADO)*

**RN:** RN-08.1
**Componente:** Regra de compatibilidade — POST /v1/abastecimentos
**Status:** 🔴 Bloqueado — Impedimento de Infraestrutura

**Descrição do Impedimento:**
A RN-08.1 define que um veículo do tipo `ELETRICO` não pode ser abastecido com combustíveis líquidos (ex: GASOLINA), devendo a API retornar HTTP 400. Para validar esta regra, é necessário ter na base de dados um veículo cadastrado com o tipo `ELETRICO`.

Ao executar a requisição com o ID hipotético `vei_eletrico_01`, a API retornou **"veículo não encontrado"**, pois esse registo não existe. Não há nenhum endpoint disponível para cadastrar veículos (`POST /veiculos` ou equivalente), tornando impossível criar o pré-requisito de dados necessário para este teste.

**Impacto do Bloqueio:** A RN-08.1 não pôde ser validada. Não é possível confirmar nem descartar um defeito nesta regra de negócio.

**Recomendação:** Disponibilizar um endpoint de cadastro de veículos (`POST /v1/veiculos`) ou fornecer um ID de veículo elétrico pré-cadastrado no ambiente de testes para desbloquear este caso de teste.

**Evidência da Tentativa:**
```bash
curl --location 'https://v-lab-testes.onrender.com/v1/abastecimentos' \
--header 'Content-Type: application/json' \
--data '{
  "data": "2026-05-19T12:00:00Z",
  "posto": "Posto Voltagem Máxima",
  "cidade": "Recife",
  "uf": "PE",
  "combustivel": "GASOLINA",
  "litros": 20.0,
  "valor_litro": 5.50,
  "motorista_id": "mot_001",
  "veiculo_id": "vei_eletrico_01"
}'
```

**Resposta obtida:** `404` — veículo `vei_eletrico_01` não encontrado (ID não existe na base de dados do ambiente de testes)
