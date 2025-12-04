# 📊 Desafio Power BI - Modelagem Star Schema

## 🎯 Objetivo do Projeto

Este projeto foi desenvolvido como parte do bootcamp de Power BI, com o objetivo de criar um modelo dimensional baseado em **Star Schema** a partir da base de dados **Financial Sample** do Power BI.

O desafio consistiu em transformar uma tabela única em um modelo dimensional completo, aplicando conceitos de modelagem de dados, transformações no Power Query e funções DAX.

---

## 📁 Estrutura do Projeto

```
📦 Desafio-PowerBI-StarSchema
 ┣ 📜 Desafio_PowerBI_StarSchema.pbix
 ┣ 📷 star_schema.png
 ┗ 📄 README.md
```

---

## 🌟 Modelo Star Schema

O modelo criado segue a arquitetura **Star Schema** (esquema estrela), com uma tabela fato central conectada a múltiplas tabelas dimensão:

![Star Schema](star_schema.png)

### Estrutura das Tabelas:

**Tabela Fato:**
- `F_Vendas` - Tabela central contendo as métricas e chaves estrangeiras

**Tabelas Dimensão:**
- `D_Calendário` - Dimensão temporal criada com DAX
- `D_Produtos` - Dimensão de produtos com métricas agregadas
- `D_Produtos_Detalhes` - Detalhes complementares dos produtos
- `D_Descontos` - Informações sobre descontos aplicados

**Tabela de Backup:**
- `Financials_origem` - Cópia da tabela original (oculta)

---

## 🛠️ Processo de Construção

### 1️⃣ Preparação Inicial

**Criação da tabela de backup:**
- Duplicação da tabela `Financial Sample`
- Renomeação para `Financials_origem`
- Ocultação da tabela no modelo para uso como backup

### 2️⃣ Criação das Tabelas Dimensão

#### 📅 D_Calendário (Dimensão Temporal)

**Método:** Criada com DAX usando a função `CALENDAR()`

**Código DAX utilizado:**
```dax
D_Calendário = 
ADDCOLUMNS(
    CALENDAR(
        DATE(2013, 1, 1),
        DATE(2014, 12, 31)
    ),
    "Ano", YEAR([Date]),
    "Mês", MONTH([Date]),
    "Mês Nome", FORMAT([Date], "MMMM"),
    "Trimestre", "Q" & QUARTER([Date]),
    "Dia", DAY([Date]),
    "Dia da Semana", WEEKDAY([Date]),
    "Nome Dia Semana", FORMAT([Date], "dddd"),
    "Ano-Mês", FORMAT([Date], "YYYY-MM")
)
```

**Colunas criadas:**
- Date (coluna base)
- Ano
- Mês
- Mês Nome
- Trimestre
- Dia
- Dia da Semana
- Nome Dia Semana
- Ano-Mês

**Configuração:** Marcada como "Tabela de Datas" para habilitar funções de inteligência de tempo.

---

#### 📦 D_Produtos (Dimensão de Produtos Agregada)

**Método:** Power Query - Agrupamento de dados

**Processo:**
1. Criação de referência da tabela `Financial Sample`
2. Agrupamento por `Product` com as seguintes agregações:
   - **Média de Unidades Vendidas** (AVERAGE de Units Sold)
   - **Média do valor de vendas** (AVERAGE de Sales)
   - **Mediana do valor de vendas** (MEDIAN de Sales)
   - **Valor máximo de Venda** (MAX de Sales)
   - **Valor mínimo de Venda** (MIN de Sales)
3. Adição de coluna de índice para criar `ID_produto`
4. Reorganização das colunas

**Colunas finais:**
- ID_produto
- Produto
- Média de Unidades Vendidas
- Média do valor de vendas
- Mediana do valor de vendas
- Valor máximo de Venda
- Valor mínimo de Venda

---

#### 🔍 D_Produtos_Detalhes (Detalhes dos Produtos)

**Método:** Power Query - Seleção de colunas

**Processo:**
1. Criação de referência da tabela `Financial Sample`
2. Seleção das colunas específicas
3. Manutenção dos registros detalhados (sem agrupamento)

**Colunas:**
- Product (ID_produtos)
- Discount Band
- Sale Price
- Units Sold
- Manufacturing Price
- Índice

---

#### 💰 D_Descontos (Dimensão de Descontos)

**Método:** Power Query - Seleção e remoção de duplicatas

**Processo:**
1. Criação de referência da tabela `Financial Sample`
2. Seleção das colunas relacionadas a descontos
3. Remoção de linhas duplicadas
4. Renomeação de `Product` para `ID_produto`

**Colunas:**
- ID_produto
- Discount
- Discount Band
- Índice

---

### 3️⃣ Criação da Tabela Fato

#### 💼 F_Vendas (Tabela Fato Central)

**Método:** Power Query - Seleção de colunas + Chave primária

**Processo:**
1. Criação de referência da tabela `Financial Sample`
2. Seleção das colunas de métricas e chaves estrangeiras
3. Adição de coluna de índice `SK_ID` (chave primária única)
4. Reorganização das colunas

**Colunas:**
- SK_ID (chave primária)
- ID_Produto (chave estrangeira)
- Product
- Units Sold
- Sales Price
- Discount Band
- Segment
- Country
- Profit
- Date (chave estrangeira)

---

### 4️⃣ Criação dos Relacionamentos

**Relacionamentos criados (Star Schema):**

| Tabela Origem | Coluna Origem | Tabela Destino | Coluna Destino | Cardinalidade |
|---------------|---------------|----------------|----------------|---------------|
| F_Vendas | Date | D_Calendário | Date | N:1 |
| F_Vendas | ID_Produto | D_Produtos | ID_produto | N:1 |
| F_Vendas | ID_Produto | D_Produtos_Detalhes | Product | N:1 |
| F_Vendas | ID_Produto | D_Descontos | ID_produto | N:1 |

**Configurações:**
- Direção do filtro: Única (das dimensões para o fato)
- Todos os relacionamentos ativos
- Cardinalidade: Muitos para Um (N:1)

---

## 📚 Funções e Técnicas Utilizadas

### Funções DAX:

| Função | Utilização |
|--------|------------|
| `CALENDAR()` | Criação da tabela calendário com intervalo de datas |
| `ADDCOLUMNS()` | Adição de colunas calculadas à tabela calendário |
| `YEAR()`, `MONTH()`, `DAY()` | Extração de componentes de data |
| `QUARTER()` | Extração do trimestre |
| `FORMAT()` | Formatação de datas como texto |
| `WEEKDAY()` | Identificação do dia da semana |
| `DATE()` | Criação de valores de data |

### Transformações Power Query:

| Técnica | Aplicação |
|---------|-----------|
| Referência de tabela | Criação de novas tabelas baseadas na original |
| Duplicação de tabela | Backup da tabela original |
| Agrupar Por (Group By) | Agregação de dados em D_Produtos |
| Remover Duplicatas | Limpeza de dados em dimensões |
| Adicionar Coluna de Índice | Criação de chaves primárias (SK_ID, ID_produto) |
| Selecionar/Remover Colunas | Definição da estrutura de cada tabela |
| Renomear Colunas | Padronização de nomenclatura |

---

## ✅ Resultados Obtidos

- ✔️ Modelo dimensional Star Schema funcional
- ✔️ 1 Tabela Fato central com métricas de vendas
- ✔️ 4 Tabelas Dimensão para análises multidimensionais
- ✔️ Relacionamentos N:1 corretamente configurados
- ✔️ Tabela calendário otimizada para inteligência de tempo
- ✔️ Backup da tabela original preservado
- ✔️ Modelo pronto para criação de dashboards e relatórios

---

## 🎓 Aprendizados

Durante o desenvolvimento deste projeto, foram aplicados conceitos fundamentais de:

- **Modelagem Dimensional**: Implementação do modelo Star Schema
- **ETL com Power Query**: Transformação e preparação de dados
- **DAX**: Criação de tabelas calculadas e colunas customizadas
- **Relacionamentos**: Configuração de cardinalidade e direção de filtros
- **Boas Práticas**: Organização de dados, nomenclatura e documentação

---

## 🚀 Como Utilizar

1. Baixe o arquivo `Desafio_PowerBI_StarSchema.pbix`
2. Abra no Power BI Desktop
3. Explore o modelo na visualização de **Modelo**
4. Crie visualizações utilizando as tabelas dimensão e fato
5. Aplique funções de inteligência de tempo usando a D_Calendário

---

## 👨‍💻 Autor

Projeto desenvolvido como parte do Bootcamp de Power BI

📅 Data: Dezembro 2024

---

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais como parte de um bootcamp.

---

## 🔗 Links Úteis

- [Documentação Power BI](https://docs.microsoft.com/power-bi/)
- [DAX Guide](https://dax.guide/)
- [Power Query M Reference](https://docs.microsoft.com/powerquery-m/)

---

**⭐ Se este projeto foi útil para você, considere dar uma estrela no repositório!**









