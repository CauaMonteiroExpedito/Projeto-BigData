# Metadados do Projeto — Origem Energia

**Projeto:** Análise da Infraestrutura Computacional e Impacto Operacional
**Disciplina:** Tópicos de Big Data em Python  
**Instituição:** Estácio de Sá  
**Fonte dos dados:** Inventario.xlsx — fornecido por Marcel Santos, Coordenador de TI da Origem Energia  
**Total de registros:** 991 linhas  

---

## 1. Origem dos dados

Os dados foram extraídos do sistema de inventário de infraestrutura computacional da Origem Energia e compartilhados via planilha Excel (.xlsx) mediante assinatura de Carta de Autorização em 14 de abril de 2026. Os dados são de natureza técnica e não sensível, referentes às especificações de hardware dos equipamentos da empresa.

---

## 2. Colunas originais

| Coluna | Tipo | Descrição |
|---|---|---|
| Last inventory | datetime | Data e hora do último registro de inventário do equipamento |
| Operating system | string | Sistema operacional instalado no equipamento |
| RAM (MB) | inteiro | Quantidade de memória RAM em megabytes |
| CPU (MHz) | inteiro | Frequência do processador em megahertz |
| CPU type | string | Descrição completa do processador, incluindo fabricante, modelo, frequência e número de núcleos |
| CPU number | inteiro | Quantidade de CPUs físicas instaladas na máquina |

---

## 3. Colunas derivadas (criadas no tratamento)

| Coluna | Tipo | Descrição | Como foi criada |
|---|---|---|---|
| RAM_GB | float | Memória RAM convertida para gigabytes | RAM (MB) / 1024 |
| CPU_Family | string | Família do processador extraída da string bruta | Regex + condicionais sobre a coluna CPU type |
| Cores_per_CPU | inteiro | Número de núcleos por processador | Regex extraindo o valor dentro de [X core(s)] |
| Total_Cores | inteiro | Total de núcleos na máquina | Cores_per_CPU × CPU number |
| Machine_Profile | string | Perfil estimado do equipamento | Regras baseadas em OS e modelo de CPU |
| Impacto_Operacional | string | Indicador simulado de impacto operacional | Score calculado com base em RAM, núcleos e CPU_Family |

---

## 4. Categorias de CPU_Family

| Valor | Descrição |
|---|---|
| Core i3 | Intel Core i3 — linha de entrada |
| Core i5 | Intel Core i5 — linha intermediária |
| Core i7 | Intel Core i7 — linha de alto desempenho consumer |
| Core i9 | Intel Core i9 — linha topo de linha consumer |
| Ryzen 3 | AMD Ryzen 3 — linha de entrada |
| Ryzen 5 | AMD Ryzen 5 — linha intermediária |
| Ryzen 7 | AMD Ryzen 7 — linha de alto desempenho |
| Xeon Silver | Intel Xeon Silver — linha de servidor intermediária |
| Xeon Gold | Intel Xeon Gold — linha de servidor avançada |
| Xeon Platinum | Intel Xeon Platinum — linha de servidor topo |
| Xeon Outro | Intel Xeon sem linha identificada |
| Outro | CPU não classificada nas categorias acima |

---

## 5. Categorias de Machine_Profile

| Valor | Critério de classificação |
|---|---|
| Servidor | OS contém "Server" ou "Ubuntu" |
| Notebook | OS é Windows 11/10 e CPU possui sufixo mobile (U, H, G1, etc.) |
| Desktop | OS é Windows 11/10 e CPU não possui sufixo mobile |
| Indefinido | Não se enquadra nas regras acima |

---

## 6. Lógica do dado simulado — Impacto_Operacional

O indicador foi criado por pontuação composta:

**RAM:**
- ≥ 32 GB → 3 pontos
- 16 a 31 GB → 2 pontos
- < 16 GB → 1 ponto

**Total de núcleos:**
- ≥ 8 → 3 pontos
- 4 a 7 → 2 pontos
- < 4 → 1 ponto

**Família de CPU:**
- Xeon Platinum / Xeon Gold → 3 pontos
- Xeon Silver / Core i7 / Core i9 / Ryzen 7 / Ryzen 9 → 2 pontos
- Demais → 1 ponto

**Classificação final:**
- Score ≥ 7 → **Baixo** (máquina robusta, menor risco operacional)
- Score 5 ou 6 → **Médio**
- Score ≤ 4 → **Alto** (máquina limitada, maior risco operacional)

**Distribuição na base:** Alto: 370 | Médio: 406 | Baixo: 215

---

## 7. Arquivo de saída

**Nome:** inventario_tratado.csv  
**Localização:** store/processed/  
**Encoding:** UTF-8 com BOM (utf-8-sig) para compatibilidade com Power BI  
**Total de colunas:** 12 (6 originais + 6 derivadas)  
**Total de linhas:** 991