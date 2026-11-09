# Projetos de APIs com Python

Três projetos de análise e visualização de dados usando APIs públicas: legislativo brasileiro, tráfego aéreo em tempo real e dados espaciais da NASA.

| Projeto | API | Bibliotecas | Autenticação |
|---|---|---|---|
| [10 - Atividade Legislativa](#projeto-10---atividade-legislativa-câmara-dos-deputados) | Dados Abertos da Câmara | requests, pandas, plotly, seaborn | Não exige |
| [11 - Rastreador de Voos](#projeto-11---rastreador-de-voos-opensky-network) | OpenSky Network | requests, pandas, folium, plotly | Opcional (OAuth2) |
| [12 - Exploração Espacial](#projeto-12---exploração-espacial-nasa-open-apis) | NASA Open APIs | requests, pandas, plotly | Chave gratuita |

## Instalação

```bash
git clone <url-do-seu-repositorio>
cd <pasta-do-repositorio>
python -m venv .venv
# Windows: .venv\Scripts\activate     Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
```

Requer Python 3.9 ou superior.

---

## Projeto 10 - Atividade Legislativa (Câmara dos Deputados)

Arquivo: `projeto10_camara.py` | API: <https://dadosabertos.camara.leg.br/api/v2>

### Desafio 1 - Ranking de deputados por tema

```bash
python projeto10_camara.py ranking --tema saúde --ano 2024
```

Busca as proposições (PL, PLP, PEC, PDL) que contêm o tema, consulta os autores de cada uma e conta apenas deputados. Gera um gráfico de barras horizontal.

| Opção | Padrão | Descrição |
|---|---|---|
| `--tema` | saúde | Palavra-chave da proposição |
| `--ano` | 2024 | Ano de apresentação |
| `--top` | 15 | Quantidade de deputados no ranking |
| `--tipos` | PL PLP PEC PDL | Tipos de proposição |
| `--max-paginas` | 10 | Limite de páginas (100 itens cada) por tipo |

Saída: `ranking_<tema>_<ano>.html` e `.csv`.

### Desafio 2 - Heatmap de participação em votações

```bash
python projeto10_camara.py heatmap --ano 2024 --max-votacoes 120
```

Gera um heatmap de partido por mês com a participação em votações nominais. Saída: `heatmap_participacao_<ano>.png`.

Observações:
- Muitas votações são simbólicas e não têm votos individuais, por isso o script analisa uma **amostra** equilibrada por mês. Aumente `--max-votacoes` para mais cobertura (mais requisições e mais tempo).
- A participação é uma estimativa: votos do partido ÷ (votações nominais do mês × deputados distintos do partido).

---

## Projeto 11 - Rastreador de Voos (OpenSky Network)

Arquivo: `projeto11_opensky.py` | API: <https://opensky-network.org/api>

### Desafio 1 - Mapa de aeronaves sobre o Brasil

```bash
python projeto11_opensky.py mapa --categorias helicoptero comercial
```

Categorias disponíveis: `helicoptero`, `comercial`, `pequeno`, `planador_leve`, `drone`, `sem_info`, `outro`. Sem `--categorias`, mostra todas. Saída: `mapa_brasil.html` (uma camada por categoria, com popup de callsign, altitude e velocidade).

### Desafio 2 - Trajetória de uma aeronave

```bash
python projeto11_opensky.py trajetoria --icao24 e48b3d
```

Plota a rota no mapa e gera um gráfico de altitude (`trajetoria_<icao24>.html` e `altitude_<icao24>.html`).

O endpoint `/tracks` é experimental e pode exigir login. Alternativa que coleta posições ao vivo:

```bash
python projeto11_opensky.py trajetoria --icao24 e48b3d --poll 10 --intervalo 30
```

### Autenticação (opcional)

O uso anônimo tem poucos créditos diários. Para aumentar o limite, crie um cliente OAuth2 na sua conta OpenSky e defina:

```bash
# Windows (PowerShell)
$env:OPENSKY_CLIENT_ID="seu_id"; $env:OPENSKY_CLIENT_SECRET="seu_segredo"
# Linux/macOS
export OPENSKY_CLIENT_ID=seu_id OPENSKY_CLIENT_SECRET=seu_segredo
```

---

## Projeto 12 - Exploração Espacial (NASA Open APIs)

Arquivo: `projeto12_nasa.py` | APIs: NeoWs, APOD, EONET e Exoplanet Archive

```bash
python projeto12_nasa.py --inicio 2025-01-01 --fim 2025-01-14
```

Gera `dashboard_nasa.html` com quatro abas:

1. **APOD** - imagem (ou vídeo) astronômica do dia, com título e explicação
2. **Exoplanetas** - barras por método de descoberta (escala log)
3. **Eventos naturais** - mapa dos eventos EONET das últimas 48 horas
4. **Asteroides** - dispersão distância × velocidade no intervalo escolhido (tamanho = diâmetro, cor = potencialmente perigoso)

### Chave da NASA

Gere uma chave gratuita em <https://api.nasa.gov> e defina:

```bash
# Windows (PowerShell)
$env:NASA_API_KEY="sua_chave"
# Linux/macOS
export NASA_API_KEY=sua_chave
```

Sem chave, o script usa `DEMO_KEY`, que tem limite baixo de requisições por hora. O feed de asteroides aceita no máximo 7 dias por chamada; o script divide intervalos maiores automaticamente.

---

## Conceitos praticados

- Paginação (links `next`), retry com backoff e tratamento de erros HTTP
- Autenticação por chave de API e OAuth2
- Dados relacionais (deputado → proposição → votação) e JSON aninhado
- Coordenadas geográficas, bounding box e mapas interativos
- Escalas logarítmicas e dashboards com abas em HTML

## Limitações conhecidas

- As APIs públicas podem mudar parâmetros, limites ou exigir autenticação sem aviso.
- Os resultados do projeto 10 (desafio 2) usam amostragem e são aproximados.
- Os dados do OpenSky dependem da cobertura dos receptores ADS-B e podem ter lacunas.

## Fontes de dados

- [Dados Abertos da Câmara](https://dadosabertos.camara.leg.br)
- [OpenSky Network](https://opensky-network.org)
- [NASA Open APIs](https://api.nasa.gov) e [Exoplanet Archive](https://exoplanetarchive.ipac.caltech.edu)
