# PLANO DE TESTE — API de Combustíveis V-Lab

**Versão:** 1.0  
**Data:** 2026-05-19 
**Autor:** Emanuel Henrique  
**Projeto:** Desafio QA V-Lab  

---

## 1. Objetivo

Validar funcionalmente a API de Combustíveis (`https://v-lab-testes.onrender.com/`) em fase de homologação, garantindo que todos os critérios de aceitação definidos em `02_CRITERIOS_DE_ACEITACAO.md` (RN-01 a RN-08) sejam verificados, bugs documentados com evidências reais e riscos mapeados.

---

## 2. Escopo

### 2.1 Em Escopo

| # | Área |
|---|------|
| 1 | CRUD completo de abastecimentos (POST, GET, PUT, DELETE) |
| 2 | Validações de campos obrigatórios e tipos |
| 3 | Regras de negócio RN-01 a RN-08 |
| 4 | Filtros e paginação (RN-04) |
| 5 | Privacidade / LGPD — mascaramento de CPF (RN-03) |
| 6 | Idempotência de métodos HTTP (RN-05) |
| 7 | Endpoint de reporte de erro (RN-06) |
| 8 | Consistência temporal de datas (RN-07) |
| 9 | Coerência combustível × tipo de veículo (RN-08) |

### 2.2 Fora de Escopo

| Área | Justificativa |
|------|---------------|
| Testes de carga / stress | Fora do tempo disponível; rate limit de 15 req/min torna proibitivo |
| Autenticação / Autorização | API não exige auth nesta versão de testes |
| Testes de UI/UX | API-only, sem frontend |
| Persistência entre reinicializações | Infraestrutura de backend não documentada |
| Segurança (SQLi, XSS, etc.) | Fora do escopo do desafio |

---

## 3. Estratégia de Teste

### 3.1 Abordagem

A estratégia combina **testes exploratórios manuais** (para descoberta de comportamentos inesperados) com **testes automatizados via Python/pytest** (para cobertura sistemática e reprodutibilidade).

**Ordem de execução:**
1. Smoke test — verificar se a API responde (`GET /abastecimentos`)
2. CRUD básico — fluxo feliz completo
3. Validações de campos — limites e tipos
4. Regras de negócio — RN-01 a RN-08 em ordem
5. Casos de borda — paginação extrema, datas, IDs inválidos
6. Testes exploratórios — combinações não cobertas pelos scripts

### 3.2 Técnicas Utilizadas

| Técnica | Aplicação |
|---------|-----------|
| Partição de equivalência | Valores de `litros`, `valor_litro`, `data` |
| Análise de valor limite | `litros=0`, `valor_litro=50.00/50.01`, datas exatas de corte |
| Casos de erro (negative testing) | Campos ausentes, IDs inválidos, tipos errados |
| Testes de contrato | Verificação de campos obrigatórios na resposta |
| Testes de regressão | Reexecução após correções de bugs |

### 3.3 Ferramentas

| Ferramenta | Uso |
|------------|-----|
| Python 3.x + `requests` + `pytest` | Automação de testes |
| Postman | Exploração manual e coleção exportável |
| cURL | Evidências de linha de comando |
| GitHub | Versionamento e entrega |

---

## 4. Critérios de Entrada e Saída

### Critérios de Entrada (para início dos testes)
- API acessível em `https://v-lab-testes.onrender.com/`
- Documentação (`01_API_DOCUMENTACAO.md`) e critérios (`02_CRITERIOS_DE_ACEITACAO.md`) disponíveis
- IDs de motoristas e veículos válidos identificados (ex.: `mot_017`, `vei_008`)

### Critérios de Saída (para encerramento dos testes)
- Todos os casos de teste executados pelo menos uma vez
- Todos os bugs encontrados registrados com evidências em `RELATORIO_DE_BUGS.md`
- Taxa de execução: 100% dos casos planejados
- Taxa de sucesso documentada e justificada

---

## 5. Ambiente de Teste

| Item | Valor |
|------|-------|
| URL Base | `https://v-lab-testes.onrender.com/` |
| Autenticação | Nenhuma |
| Formato | `application/json` |
| Rate Limit | 15 requisições/minuto por IP |
| Tamanho máximo do body | 10 KB |

---

## 6. Riscos e Mitigações

| ID | Risco | Probabilidade | Impacto | Mitigação |
|----|-------|:---:|:---:|-----------|
| R-01 | API indisponível (cold start no Render) | Média | Alto | Aguardar 30s e retentar; documentar tempo de resposta |
| R-02 | IDs de motorista/veículo alterados sem aviso | Baixa | Alto | Usar `GET /abastecimentos` para confirmar IDs antes de testes |
| R-03 | Rate limit atingido durante execução de testes | Média | Médio | Controlar o volume de requisições no Postman para não exceder 15 req/min, aguardando alguns segundos entre envios em lote |
| R-04 | Ambiente de staging com dados resetados | Baixa | Médio | Scripts criam e limpam próprios dados via CRUD |
| R-05 | RN-08 sem veículos elétricos cadastrados para validar | Alta | Baixo | Documentar limitação; testar com ID hipotético e registrar comportamento real |
| R-06 | Comportamento de arredondamento dependente de linguagem do backend | Baixa | Baixo | Testar casos específicos de ponto flutuante |

---

## 7. Cronograma Estimado

| Etapa | Atividade | Duração Estimada |
|-------|-----------|------------------|
| 1 | Leitura e análise da documentação da API | 1h |
| 2 | Exploração manual (Exploratory Testing) e Smoke Test | 1h |
| 3 | Modelagem e redação do Plano e Casos de Teste | 2h |
| 4 | Execução dos testes via Postman e validação das RNs | 2h |
| 5 | Coleta de evidências e documentação dos bugs encontrados | 1h |
| 6 | Revisão final e consolidação do relatório | 1h |

---
