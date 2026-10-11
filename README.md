# SCR.data — Monitoramento Temporal de Variações Atípicas

Ferramenta experimental, desenvolvida em Python para Google Colab, destinada ao monitoramento de variações mensais nos **dados públicos agregados** do Sistema de Informações de Créditos (SCR.data), disponibilizados pelo Banco Central do Brasil (BCB).

O objetivo é **sinalizar comportamentos que mereçam investigação adicional**. Os alertas não constituem comprovação de erros de envio, transferência ou reclassificação dos mesmos contratos, nem permitem atribuir comportamentos a instituições financeiras individuais.

## Escopo da execução documentada

- **Período:** janeiro de 2025 a julho de 2026 (19 competências mensais).
- **Clientes:** pessoas jurídicas (PJ).
- **Métrica principal:** saldo da carteira ativa, em reais; visualizações em R$ bilhões.
- **Dimensões de recorte:** modalidade, submodalidade, segmento, porte, CNAE/ocupação, UF, origem e indexador.
- **Fonte:** arquivos anuais públicos do BCB, obtidos pelo notebook; o inventário registra URL, tamanho e hash SHA-256 dos arquivos consultados.

A delimitação temporal privilegia uma série sob referência normativa e metodológica mais uniforme a partir de 2025; não pretende medir os efeitos da Resolução CMN nº 4.966/2021. Eventuais alterações posteriores de leiaute ou metodologia ainda exigem verificação.

## Como executar

1. Abra `SCRdata_Monitoramento_Temporal_PJ.ipynb` no [Google Colab](https://colab.research.google.com/).
2. Execute as células em ordem, conferindo o período e os parâmetros definidos no notebook.
3. Aguarde o download ou uso do cache dos arquivos do BCB, a preparação da base PJ e a geração dos alertas.
4. Consulte as tabelas e os gráficos comparativos. Nos quatro estudos documentados, **todas as categorias disponíveis** de porte ou indexador são exibidas no mesmo recorte.
5. Obtenha `scrdata_monitoramento_temporal_resultados.zip`, gerado na pasta `scr_monitoramento/`.

A duração e os resultados de novas execuções podem variar conforme o ambiente e atualizações da fonte pública.

## Métodos e parâmetros

| Análise | Critério utilizado na execução conferida |
| --- | --- |
| Movimentos compensatórios entre portes | Variações opostas entre categorias, em meses consecutivos e recorte com demais dimensões fixas |
| Movimentos compensatórios entre indexadores | Mesmo princípio, variando o indexador |
| Compensação mínima | 95%, calculada por `200 × min(\|ΔA\|, \|ΔB\|) / (\|ΔA\| + \|ΔB\|)` |
| Valor mínimo compensado | R$ 100 milhões |
| Concentração | Participação mínima de 80% e saldo mínimo de R$ 1 bilhão |
| Reversão | Sequência de liderança A → B → A em três meses consecutivos, conforme regra do notebook |
| Valores ausentes | **Não imputar zero**; para movimentos compensatórios, comparar somente meses consecutivos com ambas as categorias publicadas |

O limiar de 80% refere-se **à participação de uma categoria no saldo do recorte**. Não se trata da regra de Pareto 80/20. Todos os limites são **exploratórios, não regulatórios** e devem ser interpretados em conjunto com as definições no código.

## Resultados da execução conferida — jan/2025 a jul/2026

Os valores abaixo foram extraídos do pacote de resultados gerado no Google Colab em **9 de outubro de 2026**. São contagens de registros de alertas produzidos pelas regras, **não quantidades de erros, contratos ou eventos independentes**.

| Método | Registros de alertas |
| --- | ---: |
| Movimentos compensatórios entre portes | 251 |
| Movimentos compensatórios entre indexadores | 37 |
| Mudanças de concentração | 62 |
| **Total das três categorias de alerta** | **350** |
| Reversões de concentração (subconjunto) | 6 |

As seis reversões são um **subconjunto** dos resultados de concentração e não devem ser somadas ao total de 350. Os resultados desta versão não são diretamente comparáveis aos de versões experimentais que imputavam zero a categorias ausentes.

### Sensibilidade do limite de compensação

| Compensação mínima | Alertas de porte | Alertas de indexador |
| --- | ---: | ---: |
| 90% | 373 | 69 |
| 95% (padrão) | 251 | 37 |
| 98% | 147 | 12 |

As contagens mudam conforme o limiar escolhido; não representam uma avaliação independente de precisão ou de falsos positivos.

## Estudos de caso reproduzíveis

O notebook produz uma tabela mensal em CSV e um gráfico PNG para cada um dos recortes abaixo, mostrando **todas as categorias publicadas** na dimensão examinada:

1. **Setembro/2025:** variação entre portes (Grande e Micro em destaque), em operações de capital de giro de até 365 dias.
2. **Janeiro/2026:** variação entre indexadores (Prefixado e Flutuantes em destaque), no recorte de clientes Micro.
3. **Março–abril/2026:** mudança de concentração e reversão entre os portes Micro e Indisponível. *Indisponível* é uma categoria de porte presente na classificação pública, não um valor criado pela ferramenta.
4. **Maio/2026:** variação entre indexadores (Prefixado e Pós-fixado em destaque), em financiamento de projeto.

Os filtros completos de cada exemplo estão definidos em `ESTUDOS_DE_CASO` no notebook. As séries exportadas preservam valores não publicados como ausentes, sem convertê-los automaticamente em zero.

## Conteúdo do ZIP de resultados

| Arquivo ou padrão | Finalidade |
| --- | --- |
| `alertas_porte.csv` | Pares de categorias de porte com movimentos compensatórios |
| `alertas_indexador.csv` | Pares de indexadores com movimentos compensatórios |
| `alertas_concentracao.csv` | Mudanças de concentração e indicadores associados |
| `resumo_alertas.csv` | Contagens por método, incluindo reversões como subconjunto |
| `sensibilidade.csv` | Contagens sob diferentes limites de compensação |
| `parametros_metodologia.json` | Recorte temporal, limites, fonte e política de ausências |
| `inventario_fontes.csv` | URLs, hashes e tamanhos dos arquivos de origem |
| `arquivos_processados.csv` | Arquivos mensais, períodos e quantidade de linhas PJ |
| `base_granular_PJ.parquet` | Base preparada para auditoria e análises adicionais |
| `*_saldos_mensais_bilhoes.csv` | Séries mensais dos quatro exemplos |
| `*_evolucao.png` | Gráficos comparativos dos quatro exemplos |

O ZIP da execução conferida contém **17 arquivos**. Para compartilhar o pacote de resultados publicamente, avalie o tamanho do arquivo Parquet e as políticas aplicáveis ao repositório.

## Limitações e evolução

- Os dados são **agregados**: não identificam contratos ou instituições financeiras individuais.
- Alertas não comprovam falhas na qualidade dos dados; podem refletir alterações econômicas, de classificação, cobertura ou publicação.
- A ausência de uma categoria em determinado mês **não equivale a saldo zero**.
- O monitoramento implementado prioriza a **carteira ativa**. Valores provisionados e quantidades de operações são possibilidades de expansão, sujeitas à avaliação da disponibilidade e comparabilidade das métricas.
- Métodos de aprendizado não supervisionado podem ser explorados em versões futuras, mas **não fazem parte dos resultados aqui apresentados**.

## Fontes

- [Portal de Dados Abertos do Banco Central do Brasil](https://dadosabertos.bcb.gov.br/)
- Arquivos SCR.data: `https://www.bcb.gov.br/pda/desig/scrdata_{ano}.zip` (anos consultados: 2025 e 2026)

**Natureza do projeto:** protótipo de pesquisa e monitoramento exploratório; não constitui ferramenta oficial ou homologada pelo BCB.
