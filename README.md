# Análise de dados – AgroTech Solutions

## 1. Visão geral

**Objetivo.** Tratar a base de vendas agropecuárias da AgroTech Solutions, produzir um relatório gerencial em SQL e transformar os resultados em 10 KPIs que ajudem a diretoria a acompanhar a empresa e a decidir. A análise será repassada a outra equipe, por isso o projeto traz o script completo e esta documentação.

**Integrantes:** Lucas Conforto 

**Disciplina:** Linguagem e projeto de banco de dados

**O que foi feito em cada parte**

| Parte | O que foi feito |
|---|---|
| Criação e carga | Criação do banco `agropecuaria_db`, da tabela `tb_vendas_agro` e carga de 75 registros importados. |
| Parte 1 – Tratamento | Remoção de registros repetidos, datas fora do período, registros de teste e correção de nomes, valores, quantidades, estados e categorias. A base passou de 75 para 68 registros. |
| Parte 2 – Relatório gerencial | 10 consultas sobre disponibilidade, fornecedores, valores, categorias, estados e período. |
| Parte 3 – KPIs | 10 indicadores com objetivo, regra de cálculo, consulta, resultado e interpretação. |
| Parte 4 – Documentação | Este README, o arquivo SQL único e os slides com os KPIs. |

**Arquivos do repositório**

```
analise_de_dados_agrotech/
├── README.md
├── analise_agrotech.sql      (criação do banco, carga, Partes 1, 2 e 3)
├── KPIs_AgroTech.pdf         (apresentação dos 10 KPIs)
```

---

## 2. Como executar

**Requisitos**
- MySQL Server 8.0 ou superior.
- Um cliente SQL, como o MySQL Workbench, ou o cliente de linha de comando `mysql`.
- O banco `agropecuaria_db` **não pode existir** antes da execução. Se já existir, rode `DROP DATABASE agropecuaria_db;` antes.

**Arquivo a executar:** `sql/analise_agrotech.sql`. Ele cria o banco, cria a tabela, carrega a base, aplica o tratamento (Parte 1) e executa as consultas das Partes 2 e 3, nessa ordem.

**Passo a passo**
1. Clone o repositório ou baixe o arquivo `sql/analise_agrotech.sql`.
2. No MySQL Workbench, abra o arquivo (File > Open SQL Script) e execute tudo (ícone do raio, ou Ctrl+Shift+Enter).
   Pela linha de comando: `mysql -u root -p < sql/analise_agrotech.sql`.
3. Confira os resultados: cada consulta devolve uma tabela de resultado, identificada pelos comentários do arquivo (`KPI 01` a `KPI 10`, `PARTE 2`, etc.).

**Observação.** O Workbench pode bloquear `UPDATE` e `DELETE` sem chave primária no `WHERE` com o erro 1175 (Safe Updates). Nesse caso, desative a opção em Edit > Preferences > SQL Editor > "Safe Updates" e reconecte.

---

## 3. Dicionário de dados

**Tabela `tb_vendas_agro`.** Guarda os registros de produtos agropecuários da AgroTech Solutions. **Cada linha representa um registro de venda de um produto**, com a quantidade registrada, o valor unitário, a data e a situação de estoque. O mesmo produto pode aparecer em mais de uma linha.

| Coluna | Tipo | Descrição | Restrições e regras | Exemplo (base tratada) |
|---|---|---|---|---|
| `id` | INT | Identificador do registro. | Chave primária (não repete). | 1 |
| `produto` | VARCHAR(100) | Nome do produto. | Grafia padronizada na Parte 1 (ex.: "Milho Safrinha"). | Milho Safrinha |
| `categoria` | VARCHAR(50) | Categoria do produto. | 6 valores: Grãos, Rações, Fertilizantes, Defensivos, Minerais e Sementes. | Grãos |
| `fornecedor` | VARCHAR(100) | Fornecedor do produto. | 6 valores: AgroSul, Campo Verde, NutriCampo, AgroMax, BioPlant e Solo Forte. | AgroSul |
| `estado` | VARCHAR(50) | Sigla do estado do registro. | 8 valores: PR, MT, GO, MS, SC, SP, RS e MG. | PR |
| `quantidade` | INT | Unidades do produto no registro. | Maior que 0 na base tratada (de 12 a 340). | 120 |
| `valor_unitario` | DECIMAL(10,2) | Valor de uma unidade, em reais. | Maior que 0 na base tratada (de R$ 42,00 a R$ 980,00). | 89.90 |
| `data_venda` | DATE | Data do registro. | Até 31/12/2025, o limite do período operacional (de 10/01/2025 a 17/05/2025 na base tratada). | 2025-01-10 |
| `estoque` | BOOLEAN | Situação do produto para comercialização. | 1 (true) ou 0 (false). | 1 |

As regras da coluna "Restrições e regras" foram aplicadas pelo tratamento da Parte 1. No `CREATE TABLE`, a única restrição declarada é a chave primária de `id`.

**Como interpretar**
- **`quantidade`:** número de unidades do registro. Nos KPIs ela é usada como a quantidade armazenada. A base não informa quanto foi efetivamente vendido em cada operação.
- **`estoque`:** `true` (1) indica produto **disponível** para comercialização; `false` (0) indica produto **indisponível**.
- **`data_venda`:** data do registro. A empresa definiu 31/12/2025 como limite do período operacional válido, e registros posteriores foram removidos.

**Valor potencial do estoque.** É o resultado de `quantidade × valor_unitario`, conforme os filtros usados. **Não é faturamento nem lucro**, pois a base não informa custos nem as vendas efetivas.

---

## 4. Resultados do tratamento e do relatório gerencial

### Parte 1 – Tratamento (75 → 68 registros)

| Passo | Correção | Linhas afetadas | Total depois |
|---|---|---|---|
| Carga inicial | Registros importados | – | 75 |
| 01 | Remoção dos registros de id 51, 52 e 53, que repetem produto, quantidade e valor dos ids 1, 2 e 3 | 3 | 72 |
| 02 | Nomes de produto corrigidos (Milhho Safrinha, Sojjja Premium, Racão Bovina 25kg, Fertilizanti NPK, Insetcida Power) | 5 | 72 |
| 03 | Nomes de fornecedor corrigidos (Campo Vrede, NutriCamp, Bio Planti) | 3 | 72 |
| 04 | `valor_unitario` inválido corrigido (R$ 1,99, R$ 99.999,99, valor negativo e zero) | 4 | 72 |
| 05 | `quantidade` inválida corrigida (valor negativo e zero) | 2 | 72 |
| 06 | Estados padronizados (PARANA → PR e SÃO PAULO → SP) | 2 | 72 |
| 07 | Categorias padronizadas (Graos → Grãos e Racao → Rações) | 2 | 72 |
| 08 | Remoção de registros com `data_venda` após 31/12/2025 (anos 2028, 2035, 2040 e 2099) | 4 | 68 |
| 09 | Remoção de registros de teste (produto XXXXX e TESTE PRODUTO, categoria ERRO e TESTE) | 0* | 68 |
| 10 | Defensivos com `valor_unitario` acima de R$ 800,00 marcados como indisponíveis (`estoque = false`) | 5 | 68 |

\* Os registros de teste (ids 74 e 75) já tinham sido removidos no passo 08, pois suas datas eram 2040 e 2099.

**Resultado:** 75 registros antes e **68 depois** (7 removidos). Dos 68, **48 estão disponíveis** e **20 indisponíveis**.

### Parte 2 – Relatório gerencial

| Item | Resultado |
|---|---|
| 01 – Registros disponíveis | **48** de 68 |
| 02 – Registros por fornecedor | Campo Verde 16; NutriCampo 14; AgroSul 11; Solo Forte 10; BioPlant 9; AgroMax 8 |
| 03 – 10 maiores valores unitários | Todos de Defensivos, de R$ 980,00 (Herbicida Platinum, SP) a R$ 599,00 (Inseticida Power, MT) |
| 04 – Valor unitário médio por categoria | Defensivos R$ 739,91; Fertilizantes R$ 296,74; Grãos R$ 136,02; Sementes R$ 117,44; Rações R$ 113,68; Minerais R$ 58,17 |
| 05 – Registros por estado | MT 14; PR 12; GO 11; MS 8; SC 7; SP 7; RS 6; MG 3 |
| 06 – Sementes e Grãos | 25 registros (9 de Sementes e 16 de Grãos) |
| 07 – Quantidade acima de 200 | 11 registros, o maior é Calcário Fino com 340 unidades (PR) |
| 08 – Valor potencial do estoque disponível | **R$ 769.784,00** |
| 09 – Registros indisponíveis por categoria | Defensivos 7; Sementes 4; Fertilizantes 3; Grãos 3; Rações 2; Minerais 1 |
| 10 – Registros entre 01/03/2025 e 31/05/2025 | 51 registros |

**Concentração de registros**
- **Fornecedores:** Campo Verde tem 16 registros (23,53%) e NutriCampo 14 (20,59%), somando 44,12% da base.
- **Estados:** MT tem 14 registros (20,59%) e PR 12 (17,65%), somando 38,24%. MG tem apenas 3 (4,41%).

---

## 5. Catálogo e resultados dos 10 KPIs

Todos os KPIs usam a tabela `tb_vendas_agro`. No arquivo SQL, cada consulta tem o comentário com o mesmo código (`KPI 01` a `KPI 10`).

**Nos percentuais**, a divisão por zero é tratada no `WHERE`: a consulta confere se o total do denominador é maior que 0. Se fosse 0, a consulta não retornaria linhas e não faria a divisão.

### KPI 01 – Registros disponíveis
- **Objetivo:** saber quantos registros estão prontos para venda.
- **Coluna usada:** `estoque`.
- **Regra de cálculo:** contagem de registros.
- **Filtro:** `estoque = true` (só disponíveis).
- **Unidade:** registros.
- **Resultado:** **48**.
- **Interpretação:** é o que a empresa pode vender agora; os outros 20 registros estão indisponíveis. Decisão: acompanhar esse número a cada reposição.

### KPI 02 – Percentual de registros disponíveis
- **Objetivo:** medir a proporção de registros prontos para venda.
- **Coluna usada:** `estoque`.
- **Regra de cálculo:** numerador ÷ denominador × 100. **Numerador:** 48 registros com `estoque = true`. **Denominador:** 68 registros (todos).
- **Filtro:** numerador só disponíveis; denominador todos os registros.
- **Unidade:** % (duas casas decimais).
- **Resultado:** **70,59%**.
- **Interpretação:** 3 em cada 10 registros (29,41%) estão indisponíveis. Decisão: definir uma meta de disponibilidade (sugestão do grupo: 80,00%) e reforçar a reposição.

### KPI 03 – Unidades disponíveis
- **Objetivo:** medir o volume físico pronto para venda.
- **Coluna usada:** `quantidade` e `estoque`.
- **Regra de cálculo:** soma de `quantidade`.
- **Filtro:** `estoque = true`.
- **Unidade:** unidades.
- **Resultado:** **5.681 unidades**.
- **Interpretação:** é o volume que pode ser vendido hoje. Decisão: usar como referência no planejamento de vendas e da reposição.

### KPI 04 – Unidades disponíveis por estado
- **Objetivo:** ver onde o estoque disponível está concentrado.
- **Colunas usadas:** `estado`, `quantidade`, `estoque`.
- **Regra de cálculo:** soma de `quantidade` por estado.
- **Filtro:** `estoque = true`.
- **Unidade:** unidades.
- **Resultado:** PR 1.777; MT 1.510; MS 768; GO 459; SC 390; MG 320; RS 265; SP 192.
- **Interpretação:** PR e MT têm 3.287 das 5.681 unidades (mais da metade) e SP tem a menor quantidade. Decisão: avaliar se o estoque de cada estado acompanha a demanda da região.

### KPI 05 – Valor potencial do estoque disponível
- **Objetivo:** estimar em reais o valor do estoque disponível.
- **Colunas usadas:** `quantidade`, `valor_unitario`, `estoque`.
- **Regra de cálculo:** soma de `quantidade × valor_unitario`.
- **Filtro:** `estoque = true`.
- **Unidade:** reais (R$).
- **Resultado:** **R$ 769.784,00**.
- **Interpretação:** é o valor potencial do estoque disponível, se todo fosse vendido. Não é faturamento nem lucro. Decisão: usar como referência do potencial de venda.

### KPI 06 – Valor potencial do estoque disponível por categoria
- **Objetivo:** comparar onde está o valor do estoque disponível.
- **Colunas usadas:** `categoria`, `quantidade`, `valor_unitario`, `estoque`.
- **Regra de cálculo:** soma de `quantidade × valor_unitario` por categoria.
- **Filtro:** `estoque = true`.
- **Unidade:** reais (R$).
- **Resultado:** Grãos R$ 338.018,00; Fertilizantes R$ 120.755,00; Defensivos R$ 101.155,00; Rações R$ 92.089,20; Minerais R$ 77.968,00; Sementes R$ 39.798,80.
- **Interpretação:** Grãos concentra 43,91% do valor potencial disponível (R$ 338.018,00 ÷ R$ 769.784,00). Decisão: dar atenção especial à gestão do estoque de Grãos.

### KPI 07 – Percentual de registros por fornecedor
- **Objetivo:** ver o peso de cada fornecedor na base.
- **Coluna usada:** `fornecedor`.
- **Regra de cálculo:** numerador ÷ denominador × 100. **Numerador:** registros do fornecedor. **Denominador:** 68 registros (todos).
- **Filtro:** nenhum (todos os registros).
- **Unidade:** % (duas casas decimais).
- **Resultado:**

| Fornecedor | Registros | Percentual |
|---|---|---|
| Campo Verde | 16 | 23,53% |
| NutriCampo | 14 | 20,59% |
| AgroSul | 11 | 16,18% |
| Solo Forte | 10 | 14,71% |
| BioPlant | 9 | 13,24% |
| AgroMax | 8 | 11,76% |

- **Interpretação:** Campo Verde e NutriCampo somam 44,12% dos registros, e AgroMax tem a menor participação. Decisão: acompanhar mais de perto os dois principais fornecedores.

### KPI 08 – Percentual de unidades disponíveis
- **Objetivo:** medir quanto do volume total de unidades está disponível.
- **Colunas usadas:** `quantidade`, `estoque`.
- **Regra de cálculo:** numerador ÷ denominador × 100. **Numerador:** 5.681 unidades com `estoque = true`. **Denominador:** 7.387 unidades de todos os registros.
- **Filtro:** numerador só disponíveis; denominador todos os registros.
- **Unidade:** % (duas casas decimais).
- **Resultado:** **76,91%**.
- **Interpretação:** é maior que o percentual de registros disponíveis (70,59%), ou seja, os indisponíveis têm volume menor. Decisão: dimensionar a reposição pelas 1.706 unidades indisponíveis.

### KPI 09 – Valor unitário médio por categoria
- **Objetivo:** comparar o preço médio das categorias.
- **Colunas usadas:** `categoria`, `valor_unitario`.
- **Regra de cálculo:** média de `valor_unitario` por categoria.
- **Filtro:** nenhum (todos os registros).
- **Unidade:** reais (R$).
- **Resultado:** Defensivos R$ 739,91; Fertilizantes R$ 296,74; Grãos R$ 136,02; Sementes R$ 117,44; Rações R$ 113,68; Minerais R$ 58,17.
- **Interpretação:** o valor médio de Defensivos é mais de 12 vezes o de Minerais. Decisão: cuidar mais do estoque de Defensivos, pois cada unidade que falta pesa mais.

### KPI 10 – Registros indisponíveis por categoria
- **Objetivo:** mostrar em quais categorias faltam mais registros.
- **Colunas usadas:** `categoria`, `estoque`.
- **Regra de cálculo:** contagem de registros por categoria.
- **Filtro:** `estoque = false` (só indisponíveis).
- **Unidade:** registros.
- **Resultado:** Defensivos 7; Sementes 4; Fertilizantes 3; Grãos 3; Rações 2; Minerais 1 (total 20).
- **Interpretação:** Defensivos e Sementes somam 11 dos 20 registros indisponíveis (55,00%). Decisão: começar a reposição por essas duas categorias.

---

## 6. Conclusão gerencial

Os três KPIs mais relevantes para a diretoria são:

1. **KPI 10 – Registros indisponíveis por categoria.** Mostra onde está o problema: Defensivos (7) e Sementes (4) concentram 55,00% dos registros indisponíveis. Apoia a decisão de onde repor primeiro.
2. **KPI 02 – Percentual de registros disponíveis.** Resume a situação geral: 70,59% dos registros estão disponíveis e 29,41% estão fora de venda. Apoia a definição de uma meta de disponibilidade e o acompanhamento da reposição.
3. **KPI 06 – Valor potencial do estoque disponível por categoria.** Mostra onde está o valor do estoque: Grãos tem 43,91% dos R$ 769.784,00 disponíveis. Apoia a decisão de proteger o estoque dessa categoria.

**Recomendações:** repor primeiro Defensivos e Sementes; definir uma meta de disponibilidade (a sugestão do grupo é 80,00%, pois a empresa não forneceu meta); proteger o estoque de Grãos; e acompanhar Campo Verde e NutriCampo, que somam 44,12% dos registros.
