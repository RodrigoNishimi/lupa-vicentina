# Lupa Vicentina: Dashboard COOP Clima São Vicente

**Link do repositório Github:** https://github.com/RodrigoNishimi/lupa-vicentina.git

**Link do site:** https://rodrigonishimi.github.io/lupa-vicentina/

Painel web 100% estático de utilidade pública para os moradores de São Vicente (SP): demografia, escolas e unidades de saúde, bairro a bairro. Não há backend nem banco de dados — um script de ETL em Python lê as planilhas, trata os dados e gera o index.html com todos os dados JSON embutidos, pronto para hospedar no GitHub Pages.

O index.html carrega junto a pasta `assets/oficinas/` (imagens da seção das Oficinas Participativas de Avaliação, referenciadas por caminho relativo) — publique as duas coisas.

Além dos números, o painel é construído para responder cinco perguntas ao visitante: **o que estou vendo** (abertura, selos de escopo e subtítulos), **por que isso importa** (bloco de contexto por seção), **como interpretar** ("Como ler" nos gráficos, glossário e notas metodológicas), **de onde vieram os dados** (legenda de fonte em cada elemento e tabela de fontes) e **posso reutilizar** (licença, download em CSV/JSON e citação pronta).

## Estrutura do Projeto

```text
├── assets/
│   ├── logo_branco_verde.png            # Logo do projeto embutida no header
│   └── oficinas/                        # Imagens da seção OPA! (caminho relativo, não embutidas)
├── scripts/                             # Scripts de limpeza e obtenção de dados
│   ├── obter_coordenadas.py
│   ├── process_bases.py
│   └── process_census_data.py
├── dados.zip                            # Arquivo com os dados utilizados no Dashboard
├── etl.py                               # Script de ETL: gera o dashboard final
├── template.html                        # Template do dashboard com marcadores de injeção
├── index.html                           # ARQUIVO FINAL gerado pelo ETL (não editar à mão — edite o template.html)
├── requirements.txt                     # Dependências do projeto
├── LICENSE                              # GNU GPL v3.0
└── README.md
```

## Configuração do Ambiente

1. Extraia os dados
Descompacte o arquivo dados.zip na raiz do projeto. Isso populará a pasta dados/ com os arquivos CSV necessários.

2. Crie e ative o ambiente virtual (venv)
No Windows:
```bash
python -m venv .venv
.venv\Scripts\activate
```

No Linux/Mac:
```bash
python -m venv .venv
source .venv/bin/activate
```

3. Instale as bibliotecas necessárias
```bash
pip install -r requirements.txt
```

## Como Executar e Atualizar

1. Processar os dados base (Opcional / Se houver atualizações)
Caso precise rodar a limpeza dos dados novamente:
```bash
python scripts/process_census_data.py
```

2. Gerar o dashboard
Com as planilhas finalizadas em dados/, execute o ETL para criar o HTML:
```bash
python etl.py
```

Saída esperada:
> OK: index.html gerado.
> bairros do censo: 29 | escolas: 241 | saúde: 427
> bairros extras (sem censo): 15
> registros sem coordenada válida (fora do mapa): 47

3. Visualizar localmente
Basta abrir o index.html no navegador, ou iniciar um servidor local:
```bash
python -m http.server 8123
```

## Publicar no GitHub Pages

1. Faça commit do index.html gerado no repositório.
2. No GitHub, vá em Settings -> Pages e aponte para a branch/pasta que contém o index.html.
3. O painel ficará disponível no link gerado pelo GitHub Pages.

## O que o painel oferece

* Abertura "O que é a Lupa Vicentina?" — o que o visitante está vendo, por que aquilo importa e como usar em 3 passos, com contadores preenchidos a partir dos próprios dados e uma faixa de atalhos que funciona como índice das seções.
* Filtro global por bairro (dropdown) — KPIs, gráficos, mapa e tabelas atualizam sem recarregar a página.
* KPIs: população, renda média (salários mínimos), densidade (hab/ha), escolas ativas, capacidade estimada de matrículas, unidades de saúde e unidades de saúde por 10 mil habitantes.
* Gráficos (Chart.js): composição populacional por cor/raça, escolas por categoria administrativa, oferta de ensino por etapa/modalidade, porte das escolas, serviços de saúde disponíveis e cobertura SUS.
* Mapa interativo (Leaflet + OpenStreetMap): escolas públicas (laranja), escolas privadas (azul), saúde SUS (verde) e demais unidades de saúde (marrom), com controle de camadas e popups de detalhes.
* Oficinas Participativas de Avaliação (OPA!): seção editorial de escopo fixo (não responde ao filtro de bairro) com o processo de escuta territorial do projeto — números (11 oficinas, 361 participantes, 968 pontos mapeados), abordagem passado/presente/futuro, linha do tempo da construção, mapa das 6 áreas com as facilitadoras, registros fotográficos e o mapa colaborativo resultante.
* Rankings: TOP 10 bairros por escolas, por unidades de saúde e por densidade demográfica.
* Tabelas de detalhamento em abas (Escolas / Saúde), com busca por nome, endereço ou bairro.
* Camada editorial de leitura: um bloco "Por que isso importa" no topo de cada seção e um "Como ler" dentro de cada gráfico (por que o radar passa de 100%, por que as fatias da rosca se comparam entre anéis, etc.).
* "Como ler este painel": cuidados de leitura, glossário dos termos usados nos indicadores e notas metodológicas — a versão para o leitor das decisões de ETL listadas mais abaixo.
* "Dados abertos e reuso": data de geração da versão, tabela de fontes com licença de cada conjunto, declaração de licença do projeto, citação pronta com botão de copiar e **download dos dados tratados em CSV (por conjunto) ou JSON**, gerados no próprio navegador a partir do JSON embutido — sem servidor.

## Decisões de Tratamento de Dados (ETL)

| Problema encontrado nos dados | Tratamento aplicado |
| :--- | :--- |
| Nomes de bairro divergentes | Normalização (maiúsculas, sem acento, abreviações expandidas), tabela de apelidos e casamento por prefixo para nomes truncados. |
| Coordenadas da base de saúde sem ponto decimal | Divisão sucessiva por 10 até a magnitude correta. |
| Pontos geocodificados fora de São Vicente | Excluídos do mapa (continuam nas tabelas); a legenda informa quantos ficaram de fora. |
| 23 escolas paralisadas | Ficam nas tabelas com status "Paralisada", mas fora dos KPIs e gráficos. |
| Porte "Mais de 1000 matrículas" (faixa aberta) | Capacidade contabilizada como 1.000 (piso da faixa) — por isso o KPI mostra valor aproximado. |
| Bairros presentes só nas bases de escolas/saúde | Aparecem no dropdown em "Outros bairros (sem dados do censo)": mapa e tabelas funcionam; indicadores demográficos exibem "-". |
| Renda média da cidade | Média por bairro ponderada pela população. |

Essas decisões aparecem para o usuário final em "Como ler este painel" → Notas metodológicas. Ao alterar qualquer uma delas no `etl.py`, atualize também o texto correspondente no `template.html`.

## Fontes de Dados e Licenças

| Conjunto | Fonte | Referência | Condição de uso |
| :--- | :--- | :--- | :--- |
| População, cor/raça, renda e área por bairro | [Censo Demográfico — IBGE](https://censo2022.ibge.gov.br/) | 2022 | Dado público — uso livre com citação |
| Escolas | [Catálogo de Escolas — INEP/MEC](https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos) | Extração de jul/2026 | Dados abertos federais — uso livre com citação |
| Unidades de saúde | [CNES — DATASUS/MS](https://cnes.datasus.gov.br/) | Extração de jul/2026 | Dados abertos federais — uso livre com citação |
| Mapa e busca de endereços | [OpenStreetMap e Nominatim](https://www.openstreetmap.org/copyright) | Contínua | ODbL — atribuição obrigatória |
| Fotos da cidade | [Wikimedia Commons](https://commons.wikimedia.org/) | Ver cada imagem | Creative Commons — crédito na legenda |
| Oficinas OPA!, notícias e texto histórico | Acervo do COOP Clima São Vicente; Prefeitura e imprensa regional | 2025–2026 | Uso mediante crédito |

O painel é uma fotografia do momento da extração: INEP e CNES atualizam suas bases continuamente. A data da versão publicada vem do campo `gerado_em`, preenchido pelo `etl.py` e exibido na seção "Dados abertos e reuso" e no rodapé.

## Licença

O código, os dados tratados e os textos próprios da Lupa Vicentina são distribuídos sob a **GNU General Public License v3.0** — texto integral em [LICENSE](LICENSE). É livre copiar, adaptar e redistribuir — inclusive adaptando o painel para outra cidade — desde que se cite a origem e os trabalhos derivados mantenham a mesma licença, com o código-fonte disponível.

Os dados de origem seguem as licenças dos respectivos produtores, listadas na tabela acima.

O rodapé e a seção "Dados abertos e reuso" do painel publicado apontam para a mesma licença.

## Identidade Visual

* Logo: A logo oficial é embutida no cabeçalho como data URI (lida pelo etl.py da pasta assets/), substituindo o título em texto. Integrada com fundo verde #007a4a.
* Cores institucionais: Verde #007a4a, marrom #874a33, azul claro #00a3e0, laranja #e87722; fundo off-white #f9f3f0.
* Tipografia: Londrina Solid (títulos) e Roboto (textos, tabelas e KPIs) via Google Fonts.
* Bibliotecas (CDN): Chart.js 4 e Leaflet 1.9 (requer internet para visualização correta).
