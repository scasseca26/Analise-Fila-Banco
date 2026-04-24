# Analise-Fila-Banco
Projeto de análise de filas em um banco utilizando Excel, com foco na otimização do tempo de atendimento e eficiência do serviço.
![Excel](https://img.shields.io/badge/Ferramenta-Excel-217346?style=flat&logo=microsoftexcel&logoColor=white)
![Power Query](https://img.shields.io/badge/ETL-Power%20Query-F2C811?style=flat&logo=microsoft&logoColor=black)
![Status](https://img.shields.io/badge/Status-Concluído-brightgreen)

---

## Resumo Executivo

Este projecto analisa **15.969 senhas** registadas no sistema de filas de um banco fictício em Luanda, ao longo de **3 meses de operação** (Janeiro, Fevereiro e Março de 2025), distribuídas por **10 agências** localizadas em zonas como Talatona, Maianga, Viana, Cazenga, Kilamba, Benfica, Samba, Miramar, Rangel e Ingombota. A análise revela uma **taxa de desistência global de 15,8%**, equivalente a 2.522 clientes que abandonaram a fila sem ser atendidos, com a agência da **Maianga** a registar a maior espera média (61,5 min) e a **Viana** a maior taxa de desistência (16,8%). O **serviço de Transferência** é o que mais afasta clientes antes do atendimento, com 17,8% de desistência. A análise de prioridades revela um problema operacional crítico: os clientes **Idosos** esperam mais do que os clientes **Normais**, indicando que a política de prioridade não está a ser aplicada na prática dentro do Sistema de Gestão de Fila. A tendência geral é positiva, a taxa de desistência desceu de 16,4% em Janeiro para 15,4% em Março, mas a espera média de **59,5 minutos** continua muito elevada. O projecto foi desenvolvido inteiramente no **Microsoft Excel**, com limpeza e transformação de dados via **Power Query** e um Dashboard interactivo com filtros por Mês, Dia da Semana, Tipo de Serviço e Agência.

---

## Problema do Negócio

O Director de Operações do Banco necessitava de compreender o desempenho do sistema de atendimento nas suas agências em Luanda, com o objectivo de identificar os principais pontos de ineficiência, perceber o comportamento dos clientes nas filas e propor acções concretas de melhoria operacional.

O Gestor levantou as seguintes questões organizadas em três visões analíticas:

**Visão de Eficiência**
1. Qual a agência que mais prejudica os clientes?
2. Em que dias e horas o banco fica mais sobrecarregado?

**Visão de Serviço**

3. Qual o serviço que mais afasta clientes antes de serem atendidos?
4. A qualidade do atendimento está a melhorar ou a piorar ao longo dos meses?

**Visão de Pessoas**

5. Os clientes prioritários estão realmente a ser atendidos mais rápido?
6. Quais os funcionários com desempenho abaixo da média?

---

## Contexto

O dataset foi gerado pelo **Claude (IA da Anthropic)** com o objectivo de simular um ambiente bancário realista, contendo problemas reais de qualidade de dados — incluindo duplicados, valores nulos, erros de digitação, formatos de data inconsistentes, valores negativos e inconsistências de negócio. O dataset original continha **20.300 registos** e foi trabalhado numa **única tabela** no Power Query, sem decomposição em Esquema Star.

> **Fonte dos dados:** Gerado artificialmente via Claude (Anthropic) para fins de aprendizagem e prática de análise de dados.

---

## Premissas da Análise

- Os dados foram tratados e modelados no **Power Query** do Excel.
- O dataset original (`banco_filas.csv`) foi importado como tabela base e carregado directamente na folha `banco_filas`, servindo como única fonte de dados para todas as análises.
- O horário de funcionamento bancário considerado foi das **08h00 às 15h30**, apenas em **dias úteis** (Segunda a Sexta-feira).
- Registos fora deste horário e com dias não úteis foram filtrados e excluídos no Power Query.
- Tempos de espera **negativos** ou **superiores a 300 minutos** foram tratados como nulos e excluídos dos cálculos de média.
- O threshold de desempenho dos funcionários foi definido como **10% acima da média geral** de tempo de atendimento (26,4 × 1,1 = 29,0 minutos).
- Uma desistência é classificada quando `status_atendimento = "Desistiu"`.

---

## Transformações no Power Query

### 1 — Importação e Dimensão Inicial

- Importação do ficheiro `banco_filas.csv` com **20.300 registos** e **16 colunas**.
- Verificação dos tipos de dados de cada coluna: `data_atendimento` para **Data**, `hora_chegada`, `hora_atendimento` e `hora_fim` para **Hora**, campos numéricos para **Número Inteiro**.

### 2 — Remoção de Duplicados

- Aplicação de **Remover Duplicados** com selecção de todas as colunas.
- **300 linhas duplicadas** removidas → dataset reduzido para **20.000 registos**.

### 3 — Normalização de Texto

- Remoção de espaços com **Cortar** nas colunas `agencia`, `tipo_servico`, `status_atendimento` e `prioridade`.
- Padronização de valores com **Substituir Valores**:

| Coluna | Erros corrigidos |
|--------|-----------------|
| `agencia` | `talatona` → `Talatona`, `MAIANGA` → `Maianga`, ` Cazenga` → `Cazenga`, `Viana ` → `Viana`, `Kilamba ` → `Kilamba` |
| `tipo_servico` | `Deposito`, `DEPOSITO`, `deposito`, `Depósit0`, `Depósito ` → `Depósito` / `Credito`, `CRÉDITO` → `Crédito` / `Tranferência` → `Transferência` / `Actualizacao de Dados` → `Actualização de Dados` / `pagamento de serviços` → `Pagamento de Serviços` |
| `prioridade` | `IDOSO` → `Idoso`, `gestante` → `Gestante`, ` Normal` → `Normal`, `Empresa ` → `Empresa` |

### 4 — Unificação de Formatos de Data

- A coluna `data_atendimento` continha **4 formatos diferentes**: `DD/MM/AAAA`, `AAAA-MM-DD`, `DD-MM-AAAA` e `AAAA/MM/DD`.
- Tipo da coluna alterado para **Data**, unificando automaticamente o formato.

### 5 — Tratamento de Valores Nulos

| Coluna | Nulos | Acção |
|--------|-------|-------|
| `tipo_servico` | 604 | Substituídos por `"Não Identificado"` |
| `hora_chegada` | ~1.046 | Mantidos — coluna `flag_hora_chegada` criada |
| `funcionario_id` | 813 | Substituídos por `"Não Atribuído"` |
| `observacoes` | ~8.000 | Coluna eliminada por não ter relevância analítica |

### 6 — Colunas de Flags e Controlo de Qualidade

Foram criadas colunas condicionais para identificar e documentar os problemas de dados:

| Coluna | Lógica | Valores possíveis |
|--------|--------|-------------------|
| `flag_qualidade` | Tempos negativos ou absurdos | `"Espera negativa"`, `"Espera absurda"`, `"Atendimento negativo"`, `"OK"` |
| `flag_negocio` | `Desistiu` com `hora_fim` preenchida | `"Inconsistente"`, `"OK"` |
| `flag_hora_fim` | Nulos em `hora_fim` por contexto | `"Normal (desistência)"`, `"Em falta"`, `"Inconsistente"`, `"OK"` |
| `flag_dia_semana` | Dias não úteis (Sábado e Domingo) | `"Fora do dia útil"`, `"OK"` |
| `flag_horario` | Horas fora das 08h00–15h30 | `"Fora do horário"`, `"OK"` |

### 7 — Filtro de Registos Inválidos

- Aplicação de filtros nas colunas `flag_dia_semana = "OK"` e `flag_horario = "OK"`.
- **4.031 registos eliminados** (dias não úteis e horas fora do horário bancário).
- Dataset final carregado: **15.969 registos**.

### 8 — Colunas Analíticas Adicionais

| Coluna | Descrição |
|--------|-----------|
| `contagem` | Valor fixo `1` em todas as linhas — base para contagens nas Tabelas Dinâmicas |
| `é_desistencia` | `1` se `status = "Desistiu"`, `0` caso contrário — base para taxa de desistência |
| `intervalo_hora` | Agrupamento de `hora_chegada` em intervalos de 1 hora (`"08h-09h"`, `"09h-10h"`, ...) |
| `mes_nome` | Nome do mês extraído de `data_atendimento` |
| `dia_semana` | Nome do dia da semana extraído de `data_atendimento` |

## Registo de Limpeza

| Problema | Quantidade | Acção Tomada |
|----------|------------|--------------|
| Linhas duplicadas | 300 | Removidas |
| Dias não úteis e horas fora do horário | 4.031 | Filtrados e excluídos |
| Erros em `tipo_servico` | 19 variações | Padronizadas via Substituir Valores |
| Erros em `agencia` | 5 variações | Padronizadas via Substituir Valores |
| Erros em `prioridade` | 4 variações | Padronizadas via Substituir Valores |
| Nulos em `tipo_servico` | 604 | Substituídos por "Não Identificado" |
| Nulos em `funcionario_id` | 813 | Substituídos por "Não Atribuído" |
| Tempos negativos e absurdos | ~800 | Substituídos por null |
| Inconsistências de negócio | — | Documentadas em colunas de flag |
| Dataset final carregado | **15.969 linhas** | Prontas para análise |

---

## Estratégia da Solução

### Passo 1 — Resumo do Contexto em Pergunta Aberta
> *O sistema de atendimento por senhas do banco está a funcionar de forma eficiente para os clientes e para a operação?*

### Passo 2 — Transformação em Perguntas Fechadas
> - Qual agência tem maior tempo de espera e maior taxa de desistência?
> - Quais os dias e horários com maior sobrecarga de clientes?
> - Qual o serviço com maior taxa de abandono antes do atendimento?
> - A taxa de desistência e o tempo de espera estão a melhorar mês a mês?
> - Os clientes com prioridade (Idosos e Gestantes) estão a ser atendidos antes dos clientes Normais?
> - Quais funcionários têm tempo de atendimento acima do threshold definido?

### Passo 3 — Hipóteses Analíticas

- H1: Existem agências com desempenho significativamente abaixo da média do banco.
- H2: O volume de clientes concentra-se em dias e horários específicos da semana.
- H3: Serviços mais complexos (Crédito, Abertura de Conta) geram mais desistências por terem maior tempo de atendimento.
- H4: A qualidade do atendimento está a melhorar gradualmente ao longo dos 3 meses.
- H5: A política de prioridade não está a ser aplicada de forma consistente em todos os balcões.
- H6: Existe uma variação significativa de desempenho entre os funcionários do banco.

### Passo 4 — Priorização das Hipóteses Analíticas

| Prioridade | Hipótese | Justificativa |
|------------|----------|---------------|
| Alta | H5 — Política de prioridade | Impacto directo em clientes vulneráveis (Idosos e Gestantes) |
| Alta | H1 — Agências com baixo desempenho | Orienta acções correctivas imediatas por localização |
| Alta | H3 — Serviços com mais desistências | Permite redesenho do processo de atendimento |
| Média | H2 — Dias e horários de pico | Permite optimização de escalas de funcionários |
| Média | H4 — Tendência de melhoria | Avalia o impacto de medidas já implementadas |
| Baixa | H6 — Desempenho por funcionário | Orienta necessidades de formação e apoio |

---

## Insights da Análise

### Visão de Eficiência

**Agência que mais prejudica os clientes**
A agência da **Maianga** regista o maior tempo médio de espera do banco, **61,5 minutos**, acima da média geral de 59,5 minutos. A agência de **Viana**, apesar de não ter a maior espera, apresenta a taxa de desistência mais alta, **16,8%** dos clientes abandonam a fila sem ser atendidos. Estes dois problemas têm naturezas diferentes: na Maianga os clientes esperam mas ficam; em Viana os clientes desistem mais cedo. O próximo passo seria analisar qual o tipo de serviço mais procurado em Viana e se coincide com os serviços de maior tempo de atendimento.

<img width="586" height="290" alt="Fila1" src="https://github.com/user-attachments/assets/bdc0fdde-7e31-4e25-be6d-86e53400386e" />


**Dias e horários de sobrecarga**
As **Segundas-feiras** concentram o maior volume de senhas, reflexo dos assuntos acumulados durante o fim de semana. A **Sexta-feira**, com apenas 2.716 senhas (o dia com menos volume), apresenta a maior taxa de desistência, **16,7%**, o que sugere menor capacidade operacional no final da semana. O pico de chegadas ocorre entre as **10h e as 11h**, indicando que os clientes chegam após resolverem outros compromissos matinais. Reforçar os balcões entre as 10h e as 13h, especialmente às Segundas e Sextas-feiras, teria o maior impacto na redução de desistências.

<img width="745" height="433" alt="Fila2" src="https://github.com/user-attachments/assets/0508f44a-2b19-44fc-a115-88be2e56be3c" />


### Visão de Serviço

**Serviço que mais afasta clientes**
O serviço de **Depósito** é o mais solicitado no banco. No entanto é o serviço de **Transferência** que apresenta a maior taxa de desistência, **17,8%**. É relevante notar que o serviço de **Crédito** tem a maior espera média mas uma taxa de desistência inferior, o que indica que os clientes toleram esperar mais quando o assunto é financeiramente importante. A desistência elevada nas Transferências pode indicar que os clientes percepcionam o processo como demorado face à alternativa de usar canais digitais.

<img width="1091" height="244" alt="Fila3" src="https://github.com/user-attachments/assets/0e982d81-2cd9-4366-adb0-4eb5c3fc7777" />


**Evolução da qualidade ao longo dos 3 meses**
A taxa de desistência está a diminuir consistentemente de **16,4% em Janeiro** para **15,4% em Março**, o que sugere uma melhoria gradual na qualidade do atendimento. Março é o mês com maior volume de senhas (5.489) e ao mesmo tempo a menor taxa de desistência, o que é um sinal positivo. No entanto a espera média de **59,5 minutos** mantém-se muito elevada ao longo dos três meses.

<img width="1092" height="124" alt="fILA4" src="https://github.com/user-attachments/assets/472c84a4-c82f-4731-b6d5-ec981017197c" />


### Visão de Pessoas

**Política de prioridade não está a ser aplicada**
Os clientes **Idosos** apresentam a maior espera média de todas as prioridades **61,4 minutos**, superando os clientes Normais que esperam 59,8 minutos. A diferença entre Gestantes (56,7 min) e clientes Normais (59,8 min) é de apenas 3,1 minutos, insuficiente para considerar que existe uma prioridade efectiva. As taxas de desistência são praticamente idênticas entre todas as prioridades (15,6% a 15,9%), confirmando que o sistema de prioridades não está a gerar qualquer diferença perceptível na experiência do cliente.

<img width="1092" height="148" alt="fILA5" src="https://github.com/user-attachments/assets/c88f9d83-a66b-421d-8292-b4ef8ee03bf8" />


**Desempenho dos funcionários**
Os 50 funcionários operam maioritariamente entre 23 e 29 minutos por atendimento, próximo da média geral de 26,4 minutos. Com um threshold de 10% acima da média (29,0 minutos), 4 funcionários merecem atenção, **FUN0010** (31,6 min), **FUN0049** (30,0 min), **FUN0037** (29,9 min) e **FUN0047** (29,8 min). O caso mais preocupante é o **FUN0016** que combina tempo acima da média (28,3 min) com a segunda maior taxa de desistência (18,5%), um possível sinal de sobrecarga. O **FUN0031** é a referência positiva com a menor média de atendimento (22,9 min) e a menor taxa de desistência (14,0%).

<img width="1090" height="1249" alt="Generated Image April 24, 2026 - 11_58AM" src="https://github.com/user-attachments/assets/5b71ff7a-e594-4bd8-88a2-288526d67642" />

---

## Resultados

<img width="1860" height="780" alt="Sistema Fila" src="https://github.com/user-attachments/assets/03035875-c5b8-4952-afc7-e99bb4209c37" />

Com base na análise, é possível concluir que:

- A taxa de desistência global de **15,8%** representa **2.522 clientes** que abandonaram a fila sem ser atendidos em 3 meses, um impacto operacional e de reputação significativo para o banco.
- O reforço operacional deve ser prioritário na agência de **Viana** (maior desistência) e na **Maianga** (maior espera), e nos horários entre as **10h e as 13h** de Segunda e Sexta-feira.
- O serviço de **Transferência** deve ser revisto em termos de processo ou direccionado para canais digitais, dado apresentar a maior taxa de abandono.
- A política de prioridade do banco **existe no sistema mas não existe na prática**, os Idosos esperam mais do que os clientes Normais, o que exige uma revisão do processo operacional dos balcões.
- A tendência de melhoria de Janeiro para Março é positiva, mas a espera média de **59,5 minutos** continua muito acima do aceitável para uma experiência bancária de qualidade.

---


## Estrutura do Projecto

```
Analise-Fila-Banco
│
├──  Dados
│   └── banco_filas.csv                # Dataset original gerado pelo Claude (Anthropic)
├── Analise
│   └── Analise Fila Banco.xlsx        # Ficheiro Excel com limpeza, análises e dashboard
└── README.md                          # Documentação do projecto
```

---

## Ferramentas Utilizadas

- **Microsoft Excel** — Tabelas Dinâmicas, campos calculados, Dashboard interactivo com Segmentadores de Dados
- **Power Query** — Extracção, limpeza, transformação e carregamento dos dados (ETL)

---

##  Autor

**Santiago Casseca**

[LinkedIn](https://linkedin.com/in/santiago-casseca) • [GitHub](https://github.com/santiago-casseca)
