# Anlise-de-dados-COVID-19-Dashboard
COVID Dashboard
1. Pandemia Coronavírus 2019
1.1 O COVID
O COVID-19 é uma doença causada pelo coronavírus SARS-CoV-2, que foi identificado pela primeira vez na cidade de Wuhan, na China, em dezembro de 2019. A doença é altamente contagiosa e se espalhou rapidamente pelo mundo, sendo declarada uma pandemia pela Organização Mundial da Saúde (OMS) em março de 2020.

A disponibilidade de dados sobre a evolução da pandemia no tempo em uma determinada região geográfica é fundamental para o seu combate! Este projeto busca construir um dashboard de dados para exploração e visualização interativa de dados sobre o avanço de casos e da vacinação do Brasil. O processamento de dados está neste link e o dashboard, neste link.

1.2 Dados
Os dados sobre casos da COVID-19 são compilados pelo centro de ciência de sistemas e engenharia da universidade americana John Hopkins (Link). Os dados são atualizados diariamente deste janeiro de 2020 com uma granularidade temporal de dias e geográfica de regiões de países (estados, condados, etc.). O website do projeto pode ser acessado neste link enquanto os dados, neste link. Abaixo estão descritos os dados derivados do seu processamento.

date: data de referência;
state: estado;
country: país;
population: população estimada;
confirmed: número acumulado de infectados;
confirmed_1d: número diário de infectados;
confirmed_moving_avg_7d: média móvel de 7 dias do número diário de infectados;
confirmed_moving_avg_7d_rate_14d: média móvel de 7 dias dividido pela média + + móvel de 7 dias de 14 dias atrás;
deaths: número acumulado de mortos;
deaths_1d: número diário de mortos;
deaths_moving_avg_7d: média móvel de 7 dias do número diário de mortos;
deaths_moving_avg_7d: média móvel de 7 dias dividido pela média móvel de 7 + + dias de 14 dias atrás;
month: mês de referência;
year: ano de referência.
Os dados sobre vacinação da COVID-19 são compilados pelo projeto Nosso Mundo em Dados (Our World in Data ou OWID) da universidade britânica de Oxford (link). Os dados são atualizados diariamente deste janeiro de 2020 com uma granularidade temporal de dias e geográfica de países. O website do projeto pode ser acessado neste link enquanto os dados, neste link. Abaixo estão descritos os dados derivados do seu processamento.

date: data de referência;
country: país;
population: população estimada;
total: número acumulado de doses administradas;
one_shot: número acumulado de pessoas com uma dose;
one_shot_perc: número acumulado relativo de pessoas com uma dose;
two_shots: número acumulado de pessoas com duas doses;
two_shot_perc: número acumulado relativo de pessoas com duas doses;
three_shots: número acumulado de pessoas com três doses;
three_shot_perc: número acumulado relativo de pessoas com três doses;
month: mês de referência;
year: ano de referência.
2. Pacotes e bibliotecas
Nesta sessão vamos utilizar os seguintes pacotes Python para processar os dados bruto em um formato adequado para um painel para exploração interativa de dados

[3]
0s
import math
from typing import Iterator
from datetime import datetime, timedelta

import numpy as np
import pandas as pd
3. Extração
3.1 Casos
O dado está compilado em um arquivo por dia, exemplo para 2021/12/01.

[4]
0s
cases = pd.read_csv('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/01-12-2021.csv', sep=',')
[5]
0s
cases.head()

Portanto, precisaremos iterar dentro de um intervalo de tempo definido para extraí-lo.

[6]
0s
def date_range(start_date: datetime, end_date: datetime) -> Iterator[datetime]:
  
    #Gera um intervalo de datas entre start_date e end_date.
  
  date_range_days: int = (end_date - start_date).days# Calcula o número de dias no intervalo
  for lag in range(date_range_days):# Itera sobre cada dia no intervalo
    yield start_date + timedelta(lag)# Produz (yield) uma nova data para cada dia no intervalo
[7]
0s
# Chama a função date_range com as datas especificadas e itera sobre o resultado

start_date = datetime(2021,  1,  1)
end_date   = datetime(2021, 12, 31)
De maneira iterativa, vamos selecionar as colunas de interesse e as linhas referentes ao Brasil.

[8]
1min
# Inicializa a variável 'cases' como None para armazenar os dados dos casos.
cases = None
# Define uma flag para indicar se 'cases' está vazio ou não.
cases_is_empty = True

# Itera sobre cada data no intervalo especificado.
for date in date_range(start_date=start_date, end_date=end_date):

   # Converte a data para uma string no formato 'mm-dd-yyyy'.
  date_str = date.strftime('%m-%d-%Y')
   # Constrói a URL do arquivo CSV contendo os dados para a data atual.
  data_source_url = f'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/{date_str}.csv'

   # Lê os dados do arquivo CSV para um DataFrame 'case'.
  case = pd.read_csv(data_source_url, sep=',')

   # Remove as colunas desnecessárias do DataFrame 'case'.
  case = case.drop(['FIPS', 'Admin2', 'Last_Update', 'Lat', 'Long_', 'Recovered', 'Active', 'Combined_Key', 'Case_Fatality_Ratio'], axis=1)
   # Filtra os dados para incluir apenas registros relacionados ao Brasil.
  case = case.query('Country_Region == "Brazil"').reset_index(drop=True)
   # Converte a data atual para o formato datetime e adiciona como uma coluna 'Date' no DataFrame 'case'.
  case['Date'] = pd.to_datetime(date.strftime('%Y-%m-%d'))

 
 # Verifica se 'cases' está vazio.
  if cases_is_empty:
    # Se 'cases' estiver vazio, atribui o DataFrame 'case' a 'cases'.
    cases = case
     # Atualiza a flag indicando que 'cases' não está mais vazio.
    cases_is_empty = False
  else:
    # Se 'cases' não estiver vazio, concatena 'case' com 'cases' usando o método concat do pandas.
    cases = pd.concat([cases, case], ignore_index=True)

[9]
0s
cases.query('Province_State == "Sao Paulo"').head()

3.2 Vacinação
Vamos processar os dados de vacinação da universidade de Oxford.

[13]
4s
vaccines = pd.read_csv('https://covid.ourworldindata.org/data/owid-covid-data.csv', sep=',', parse_dates=[3], infer_datetime_format=True)
[14]
0s
vaccines.head()

Vamos selecionar as colunas de interesse e as linhas referentes ao Brasil.

[15]
0s
vaccines = vaccines.query('location == "Brazil"').reset_index(drop=True)
vaccines = vaccines[['location', 'population', 'total_vaccinations', 'people_vaccinated', 'people_fully_vaccinated', 'total_boosters', 'date']]
[16]
0s
vaccines.head()

Next steps:
4. Transformação
4.1 Casos
Vamos manipular os dados para o dashboard. O foco é em garantir uma boa granularidade e qualidade da base de dados.

[20]
0s
cases.head()

Next steps:
[21]
0s
cases.shape
(9828, 6)
[22]
0s
cases.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 9828 entries, 0 to 9827
Data columns (total 6 columns):
 #   Column          Non-Null Count  Dtype         
---  ------          --------------  -----         
 0   Province_State  9828 non-null   object        
 1   Country_Region  9828 non-null   object        
 2   Confirmed       9828 non-null   int64         
 3   Deaths          9828 non-null   int64         
 4   Incident_Rate   9828 non-null   float64       
 5   Date            9828 non-null   datetime64[ns]
dtypes: datetime64[ns](1), float64(1), int64(2), object(2)
memory usage: 460.8+ KB
Começamos com o nome das colunas.

[23]
0s
cases = cases.rename(
  columns={
    'Province_State': 'state',
    'Country_Region': 'country'
  }
)

for col in cases.columns:
  cases = cases.rename(columns={col: col.lower()})
Ajustamos o nome dos estados.

[24]
0s
states_map = {
    'Amapa': 'Amapá',
    'Ceara': 'Ceará',
    'Espirito Santo': 'Espírito Santo',
    'Goias': 'Goiás',
    'Para': 'Pará',
    'Paraiba': 'Paraíba',
    'Parana': 'Paraná',
    'Piaui': 'Piauí',
    'Rondonia': 'Rondônia',
    'Sao Paulo': 'São Paulo'
}

cases['state'] = cases['state'].apply(lambda state: states_map.get(state) if state in states_map.keys() else state)
Vamos então computar novas colunas para enriquecer a base de dados.

Chaves temporais:

[25]
0s
cases['month'] = cases['date'].apply(lambda date: date.strftime('%Y-%m'))
cases['year']  = cases['date'].apply(lambda date: date.strftime('%Y'))
População estimada do estado:

[26]
0s
cases['population'] = round(100000 * (cases['confirmed'] / cases['incident_rate']))
cases = cases.drop('incident_rate', axis=1)
Número, média móvel (7 dias) e estabilidade (14 dias) de casos e mortes por estado:

[27]
0s
cases_ = None
cases_is_empty = True

def get_trend(rate: float) -> str:

  if np.isnan(rate):
    return np.NaN

  if rate < 0.75:
    status = 'downward'
  elif rate > 1.15:
    status = 'upward'
  else:
    status = 'stable'

  return status


for state in cases['state'].drop_duplicates():

  cases_per_state = cases.query(f'state == "{state}"').reset_index(drop=True)
  cases_per_state = cases_per_state.sort_values(by=['date'])

  cases_per_state['confirmed_1d'] = cases_per_state['confirmed'].diff(periods=1)
  cases_per_state['confirmed_moving_avg_7d'] = np.ceil(cases_per_state['confirmed_1d'].rolling(window=7).mean())
  cases_per_state['confirmed_moving_avg_7d_rate_14d'] = cases_per_state['confirmed_moving_avg_7d']/cases_per_state['confirmed_moving_avg_7d'].shift(periods=14)
  cases_per_state['confirmed_trend'] = cases_per_state['confirmed_moving_avg_7d_rate_14d'].apply(get_trend)

  cases_per_state['deaths_1d'] = cases_per_state['deaths'].diff(periods=1)
  cases_per_state['deaths_moving_avg_7d'] = np.ceil(cases_per_state['deaths_1d'].rolling(window=7).mean())
  cases_per_state['deaths_moving_avg_7d_rate_14d'] = cases_per_state['deaths_moving_avg_7d']/cases_per_state['deaths_moving_avg_7d'].shift(periods=14)
  cases_per_state['deaths_trend'] = cases_per_state['deaths_moving_avg_7d_rate_14d'].apply(get_trend)

  if cases_is_empty:
    cases_ = cases_per_state
    cases_is_empty = False
  else:
    cases_ = cases_.append(cases_per_state, ignore_index=True)

cases = cases_
cases_ = None
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
<ipython-input-27-aded67170a97>:38: FutureWarning: The frame.append method is deprecated and will be removed from pandas in a future version. Use pandas.concat instead.
  cases_ = cases_.append(cases_per_state, ignore_index=True)
Garantir o tipo do dado é fundamental para consistência da base de dados. Vamos fazer o type casting das colunas.

[28]
0s
cases['population'] = cases['population'].astype('Int64')
cases['confirmed_1d'] = cases['confirmed_1d'].astype('Int64')
cases['confirmed_moving_avg_7d'] = cases['confirmed_moving_avg_7d'].astype('Int64')
cases['deaths_1d'] = cases['deaths_1d'].astype('Int64')
cases['deaths_moving_avg_7d'] = cases['deaths_moving_avg_7d'].astype('Int64')
Por fim, vamos reorganizar as colunas e conferir o resultado final.

[29]
0s
cases = cases[['date', 'country', 'state', 'population', 'confirmed', 'confirmed_1d', 'confirmed_moving_avg_7d', 'confirmed_moving_avg_7d_rate_14d', 'confirmed_trend', 'deaths', 'deaths_1d', 'deaths_moving_avg_7d', 'deaths_moving_avg_7d_rate_14d', 'deaths_trend', 'month', 'year']]
[30]
0s
cases.head(n=25)

4.2 Vacinação
[31]
0s
vaccines.head()

Next steps:
[32]
0s
vaccines.shape
(1506, 7)
[33]
0s
vaccines.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1506 entries, 0 to 1505
Data columns (total 7 columns):
 #   Column                   Non-Null Count  Dtype         
---  ------                   --------------  -----         
 0   location                 1506 non-null   object        
 1   population               1506 non-null   float64       
 2   total_vaccinations       695 non-null    float64       
 3   people_vaccinated        691 non-null    float64       
 4   people_fully_vaccinated  675 non-null    float64       
 5   total_boosters           455 non-null    float64       
 6   date                     1506 non-null   datetime64[ns]
dtypes: datetime64[ns](1), float64(5), object(1)
memory usage: 82.5+ KB
Vamos começar tratando os dados faltantes, a estratégia será a de preencher os buracos com o valor anterior válido mais próximo.

[34]
0s
vaccines = vaccines.fillna(method='ffill')
Vamos também filtrar a base de dados de acordo com a coluna date para garantir que ambas as bases de dados tratam do mesmo período de tempo.

[35]
0s
vaccines = vaccines[(vaccines['date'] >= '2021-01-01') & (vaccines['date'] <= '2021-12-31')].reset_index(drop=True)
Agora, vamos alterar o nome das colunas.

[36]
0s
vaccines = vaccines.rename(
  columns={
    'location': 'country',
    'total_vaccinations': 'total',
    'people_vaccinated': 'one_shot',
    'people_fully_vaccinated': 'two_shots',
    'total_boosters': 'three_shots',
  }
)
Vamos então computar novas colunas para enriquecer a base de dados.

Chaves temporais:

[37]
0s
vaccines['month'] = vaccines['date'].apply(lambda date: date.strftime('%Y-%m'))
vaccines['year']  = vaccines['date'].apply(lambda date: date.strftime('%Y'))
Dados relativos:

[38]
0s
vaccines['one_shot_perc'] = round(vaccines['one_shot'] / vaccines['population'], 4)
vaccines['two_shots_perc'] = round(vaccines['two_shots'] / vaccines['population'], 4)
vaccines['three_shots_perc'] = round(vaccines['three_shots'] / vaccines['population'], 4)
Garantir o tipo do dado é fundamental para consistência da base de dados. Vamos fazer o type casting das colunas.

[39]
0s
vaccines['population'] = vaccines['population'].astype('Int64')
vaccines['total'] = vaccines['total'].astype('Int64')
vaccines['one_shot'] = vaccines['one_shot'].astype('Int64')
vaccines['two_shots'] = vaccines['two_shots'].astype('Int64')
vaccines['three_shots'] = vaccines['three_shots'].astype('Int64')
Por fim, vamos reorganizar as colunas e conferir o resultado final.

[40]
0s
vaccines = vaccines[['date', 'country', 'population', 'total', 'one_shot', 'one_shot_perc', 'two_shots', 'two_shots_perc', 'three_shots', 'three_shots_perc', 'month', 'year']]
[41]
0s
vaccines.tail()

5. Carregamento
5.1 Casos
Com os dados manipulados, vamos persisti-lo em disco, fazer o seu download e carrega-lo no Google Data Studio.

[42]
0s
cases.to_csv('./covid-cases.csv', sep=',', index=False)
5.2 Vacinação
Com os dados manipulados, vamos persisti-lo em disco, fazer o seu download e carrega-lo no Google Data Studio.

[44]
0s
