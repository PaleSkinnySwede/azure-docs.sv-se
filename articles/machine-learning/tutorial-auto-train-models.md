---
title: 'Regressions självstudie: automatiserad ML'
titleSuffix: Azure Machine Learning
description: I den här självstudien får du lära dig hur du skapar en maskin inlärnings modell med hjälp av automatisk maskin inlärning. Azure Machine Learning kan förbearbeta data, välja algoritm och hyperparameter på ett automatiserat sätt åt dig.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: trevorbye
ms.author: trbye
ms.reviewer: trbye
ms.date: 02/10/2020
ms.openlocfilehash: 75e61ea3f4fa6c2b346f912a9effd66ad94e7e93
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77116443"
---
# <a name="tutorial-use-automated-machine-learning-to-predict-taxi-fares"></a>Självstudie: Använd automatisk maskin inlärning för att förutse taxi priser
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

I den här självstudien använder du Automatisk maskin inlärning i Azure Machine Learning för att skapa en Regressions modell för att förutsäga priserna för NYC Taxi pris. Den här processen godkänner inlärnings data och konfigurations inställningar och upprepas automatiskt genom kombinationer av olika metoder, modeller och inställningar för en funktion för funktioner, modeller och kompatibilitetsinställningar för att komma till den bästa modellen.

![Flödesdiagram](./media/tutorial-auto-train-models/flow2.png)

I den här självstudien får du lära dig följande uppgifter:

> [!div class="checklist"]
> * Ladda ned, transformera och rensa data med Azure Open data uppsättningar
> * Träna en automatisk maskin inlärnings Regressions modell
> * Beräkna modell precision

Om du inte har en Azure-prenumeration kan du skapa ett kostnadsfritt konto innan du börjar. Prova den [kostnads fria eller betalda versionen](https://aka.ms/AMLFree) av Azure Machine Learning idag.

## <a name="prerequisites"></a>Krav

* Slutför [installations självstudien](tutorial-1st-experiment-sdk-setup.md) om du inte redan har en Azure Machine Learning arbets yta eller en virtuell dator.
* När du har slutfört installations guiden öppnar du *självstudierna/regression-automl-NYC-taxi-data/regression-Automated-ml. ipynb* Notebook med samma Notebook-Server.

Den här själv studie kursen finns också på [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials) om du vill köra den i din egen [lokala miljö](how-to-configure-environment.md#local). Kör `pip install azureml-sdk[automl] azureml-opendatasets azureml-widgets` för att hämta de nödvändiga paketen.

## <a name="download-and-prepare-data"></a>Hämta och förbereda data

Importera de nödvändiga paketen. Det öppna data uppsättnings paketet innehåller en klass som representerar varje data`NycTlcGreen` källa (till exempel) för att enkelt filtrera datum parametrar innan de hämtas.

```python
from azureml.opendatasets import NycTlcGreen
import pandas as pd
from datetime import datetime
from dateutil.relativedelta import relativedelta
```

Börja med att skapa en dataframe som innehåller taxi data. När du arbetar i en miljö som inte är Spark-miljö kan öppna data uppsättningar bara hämta en månad med data i taget med vissa klasser `MemoryError` för att undvika med stora data mängder.

För att hämta taxi data, Hämta upprepade gånger en månad i taget och innan du lägger till den för `green_taxi_df` att slumpmässigt sampla 2 000 poster från varje månad för att undvika bloating dataframe. Förhandsgranska sedan data.


```python
green_taxi_df = pd.DataFrame([])
start = datetime.strptime("1/1/2015","%m/%d/%Y")
end = datetime.strptime("1/31/2015","%m/%d/%Y")

for sample_month in range(12):
    temp_df_green = NycTlcGreen(start + relativedelta(months=sample_month), end + relativedelta(months=sample_month)) \
        .to_pandas_dataframe()
    green_taxi_df = green_taxi_df.append(temp_df_green.sample(2000))

green_taxi_df.head(10)
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Nyttolast</th>
      <th>lpepPickupDatetime</th>
      <th>lpepDropoffDatetime</th>
      <th>passengerCount</th>
      <th>tripDistance</th>
      <th>puLocationId</th>
      <th>doLocationId</th>
      <th>pickupLongitude</th>
      <th>pickupLatitude</th>
      <th>dropoffLongitude</th>
      <th>...</th>
      <th>paymentType</th>
      <th>fareAmount</th>
      <th>extra</th>
      <th>mtaTax</th>
      <th>improvementSurcharge</th>
      <th>tipAmount</th>
      <th>tollsAmount</th>
      <th>ehailFee</th>
      <th>totalAmount</th>
      <th>tripType</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>131969</th>
      <td>2</td>
      <td>2015-01-11 05:34:44</td>
      <td>2015-01-11 05:45:03</td>
      <td>3</td>
      <td>4,84</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,88</td>
      <td>40,84</td>
      <td>– 73,94</td>
      <td>...</td>
      <td>2</td>
      <td>15,00</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>16,30</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>1129817</th>
      <td>2</td>
      <td>2015-01-20 16:26:29</td>
      <td>2015-01-20 16:30:26</td>
      <td>1</td>
      <td>0,69</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,81</td>
      <td>– 73,96</td>
      <td>...</td>
      <td>2</td>
      <td>4,50</td>
      <td>1,00</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,30</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>1278620</th>
      <td>2</td>
      <td>2015-01-01 05:58:10</td>
      <td>2015-01-01 06:00:55</td>
      <td>1</td>
      <td>0,45</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,92</td>
      <td>40,76</td>
      <td>– 73,91</td>
      <td>...</td>
      <td>2</td>
      <td>4.00</td>
      <td>0,00</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>4.80</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>348430</th>
      <td>2</td>
      <td>2015-01-17 02:20:50</td>
      <td>2015-01-17 02:41:38</td>
      <td>1</td>
      <td>0,00</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,81</td>
      <td>40,70</td>
      <td>– 73,82</td>
      <td>...</td>
      <td>2</td>
      <td>12,50</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>13,80</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>1269627</th>
      <td>1</td>
      <td>2015-01-01 05:04:10</td>
      <td>2015-01-01 05:06:23</td>
      <td>1</td>
      <td>0,50</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,92</td>
      <td>40,76</td>
      <td>– 73,92</td>
      <td>...</td>
      <td>2</td>
      <td>4.00</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>5.00</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>811755</th>
      <td>1</td>
      <td>2015-01-04 19:57:51</td>
      <td>2015-01-04 20:05:45</td>
      <td>2</td>
      <td>1,10</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,72</td>
      <td>– 73,95</td>
      <td>...</td>
      <td>2</td>
      <td>6,50</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>7,80</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>737281</th>
      <td>1</td>
      <td>2015-01-03 12:27:31</td>
      <td>2015-01-03 12:33:52</td>
      <td>1</td>
      <td>0,90</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,88</td>
      <td>40,76</td>
      <td>– 73,87</td>
      <td>...</td>
      <td>2</td>
      <td>6,00</td>
      <td>0,00</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,80</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>113951</th>
      <td>1</td>
      <td>2015-01-09 23:25:51</td>
      <td>2015-01-09 23:39:52</td>
      <td>1</td>
      <td>3,30</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,72</td>
      <td>– 73,91</td>
      <td>...</td>
      <td>2</td>
      <td>12,50</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>13,80</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>150436</th>
      <td>2</td>
      <td>2015-01-11 17:15:14</td>
      <td>2015-01-11 17:22:57</td>
      <td>1</td>
      <td>1,19</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,94</td>
      <td>40,71</td>
      <td>– 73,95</td>
      <td>...</td>
      <td>1</td>
      <td>7,00</td>
      <td>0,00</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>1.75</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>9,55</td>
      <td>1,00</td>
    </tr>
    <tr>
      <th>432136</th>
      <td>2</td>
      <td>2015-01-22 23:16:33</td>
      <td>2015-01-22 23:20:13</td>
      <td>1</td>
      <td>0,65</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,94</td>
      <td>40,71</td>
      <td>– 73,94</td>
      <td>...</td>
      <td>2</td>
      <td>5.00</td>
      <td>0,50</td>
      <td>0,50</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,30</td>
      <td>1,00</td>
    </tr>
  </tbody>
</table>
<p>10 rader × 23 kolumner</p>
</div>


Nu när de ursprungliga data har lästs in definierar du en funktion för att skapa olika tidsbaserade funktioner från fältet Hämta datum/tid. Detta skapar nya fält för månads nummer, dag i månad, veckodag och veckodag och ger modellen i tidsbaserad säsongs beroende. Använd `apply()` funktionen på dataframe för att upprepa `build_time_features()` funktionen på varje rad i taxi data.

```python
def build_time_features(vector):
    pickup_datetime = vector[0]
    month_num = pickup_datetime.month
    day_of_month = pickup_datetime.day
    day_of_week = pickup_datetime.weekday()
    hour_of_day = pickup_datetime.hour

    return pd.Series((month_num, day_of_month, day_of_week, hour_of_day))

green_taxi_df[["month_num", "day_of_month","day_of_week", "hour_of_day"]] = green_taxi_df[["lpepPickupDatetime"]].apply(build_time_features, axis=1)
green_taxi_df.head(10)
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Nyttolast</th>
      <th>lpepPickupDatetime</th>
      <th>lpepDropoffDatetime</th>
      <th>passengerCount</th>
      <th>tripDistance</th>
      <th>puLocationId</th>
      <th>doLocationId</th>
      <th>pickupLongitude</th>
      <th>pickupLatitude</th>
      <th>dropoffLongitude</th>
      <th>...</th>
      <th>improvementSurcharge</th>
      <th>tipAmount</th>
      <th>tollsAmount</th>
      <th>ehailFee</th>
      <th>totalAmount</th>
      <th>tripType</th>
      <th>month_num</th>
      <th>day_of_month</th>
      <th>day_of_week</th>
      <th>hour_of_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>131969</th>
      <td>2</td>
      <td>2015-01-11 05:34:44</td>
      <td>2015-01-11 05:45:03</td>
      <td>3</td>
      <td>4,84</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,88</td>
      <td>40,84</td>
      <td>– 73,94</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>16,30</td>
      <td>1,00</td>
      <td>1</td>
      <td>11</td>
      <td>6</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1129817</th>
      <td>2</td>
      <td>2015-01-20 16:26:29</td>
      <td>2015-01-20 16:30:26</td>
      <td>1</td>
      <td>0,69</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,81</td>
      <td>– 73,96</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,30</td>
      <td>1,00</td>
      <td>1</td>
      <td>20</td>
      <td>1</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1278620</th>
      <td>2</td>
      <td>2015-01-01 05:58:10</td>
      <td>2015-01-01 06:00:55</td>
      <td>1</td>
      <td>0,45</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,92</td>
      <td>40,76</td>
      <td>– 73,91</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>4.80</td>
      <td>1,00</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>348430</th>
      <td>2</td>
      <td>2015-01-17 02:20:50</td>
      <td>2015-01-17 02:41:38</td>
      <td>1</td>
      <td>0,00</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,81</td>
      <td>40,70</td>
      <td>– 73,82</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>13,80</td>
      <td>1,00</td>
      <td>1</td>
      <td>17</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1269627</th>
      <td>1</td>
      <td>2015-01-01 05:04:10</td>
      <td>2015-01-01 05:06:23</td>
      <td>1</td>
      <td>0,50</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,92</td>
      <td>40,76</td>
      <td>– 73,92</td>
      <td>...</td>
      <td>0</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>5.00</td>
      <td>1,00</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>811755</th>
      <td>1</td>
      <td>2015-01-04 19:57:51</td>
      <td>2015-01-04 20:05:45</td>
      <td>2</td>
      <td>1,10</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,72</td>
      <td>– 73,95</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>7,80</td>
      <td>1,00</td>
      <td>1</td>
      <td>4</td>
      <td>6</td>
      <td>19</td>
    </tr>
    <tr>
      <th>737281</th>
      <td>1</td>
      <td>2015-01-03 12:27:31</td>
      <td>2015-01-03 12:33:52</td>
      <td>1</td>
      <td>0,90</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,88</td>
      <td>40,76</td>
      <td>– 73,87</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,80</td>
      <td>1,00</td>
      <td>1</td>
      <td>3</td>
      <td>5</td>
      <td>12</td>
    </tr>
    <tr>
      <th>113951</th>
      <td>1</td>
      <td>2015-01-09 23:25:51</td>
      <td>2015-01-09 23:39:52</td>
      <td>1</td>
      <td>3,30</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,96</td>
      <td>40,72</td>
      <td>– 73,91</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>13,80</td>
      <td>1,00</td>
      <td>1</td>
      <td>9</td>
      <td>4</td>
      <td>23</td>
    </tr>
    <tr>
      <th>150436</th>
      <td>2</td>
      <td>2015-01-11 17:15:14</td>
      <td>2015-01-11 17:22:57</td>
      <td>1</td>
      <td>1,19</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,94</td>
      <td>40,71</td>
      <td>– 73,95</td>
      <td>...</td>
      <td>0.3</td>
      <td>1.75</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>9,55</td>
      <td>1,00</td>
      <td>1</td>
      <td>11</td>
      <td>6</td>
      <td>17</td>
    </tr>
    <tr>
      <th>432136</th>
      <td>2</td>
      <td>2015-01-22 23:16:33</td>
      <td>2015-01-22 23:20:13</td>
      <td>1</td>
      <td>0,65</td>
      <td>Inga</td>
      <td>Inga</td>
      <td>– 73,94</td>
      <td>40,71</td>
      <td>– 73,94</td>
      <td>...</td>
      <td>0.3</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>ndet</td>
      <td>6,30</td>
      <td>1,00</td>
      <td>1</td>
      <td>22</td>
      <td>3</td>
      <td>23</td>
    </tr>
  </tbody>
</table>
<p>10 rader × 27 kolumner</p>
</div>

Ta bort några av de kolumner som du inte behöver för utbildning eller ytterligare funktions utveckling.

```python
columns_to_remove = ["lpepPickupDatetime", "lpepDropoffDatetime", "puLocationId", "doLocationId", "extra", "mtaTax",
                     "improvementSurcharge", "tollsAmount", "ehailFee", "tripType", "rateCodeID",
                     "storeAndFwdFlag", "paymentType", "fareAmount", "tipAmount"
                    ]
for col in columns_to_remove:
    green_taxi_df.pop(col)

green_taxi_df.head(5)
```

### <a name="cleanse-data"></a>Rensa data

Kör `describe()` funktionen på den nya dataframe för att se sammanfattnings statistik för varje fält.

```python
green_taxi_df.describe()
```

<div>
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Nyttolast</th>
      <th>passengerCount</th>
      <th>tripDistance</th>
      <th>pickupLongitude</th>
      <th>pickupLatitude</th>
      <th>dropoffLongitude</th>
      <th>dropoffLatitude</th>
      <th>totalAmount</th>
      <th>month_num</th>
      <th>day_of_month</th>
      <th>day_of_week</th>
      <th>hour_of_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
      <td>48000,00</td>
    </tr>
    <tr>
      <th>medelvärde</th>
      <td>1,78</td>
      <td>1,37</td>
      <td>2,87</td>
      <td>– 73,83</td>
      <td>40,69</td>
      <td>– 73,84</td>
      <td>40,70</td>
      <td>14,75</td>
      <td>6,50</td>
      <td>15,13</td>
      <td>3,27</td>
      <td>13,52</td>
    </tr>
    <tr>
      <th>standard</th>
      <td>0,41</td>
      <td>1,04</td>
      <td>2.93</td>
      <td>2,76</td>
      <td>1,52</td>
      <td>2,61</td>
      <td>1,44</td>
      <td>12,08</td>
      <td>3,45</td>
      <td>8,45</td>
      <td>1,95</td>
      <td>6,83</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1,00</td>
      <td>0,00</td>
      <td>0,00</td>
      <td>– 74,66</td>
      <td>0,00</td>
      <td>– 74,66</td>
      <td>0,00</td>
      <td>– 300,00</td>
      <td>1,00</td>
      <td>1,00</td>
      <td>0,00</td>
      <td>0,00</td>
    </tr>
    <tr>
      <th>25 %</th>
      <td>2,00</td>
      <td>1,00</td>
      <td>1.06</td>
      <td>– 73,96</td>
      <td>40,70</td>
      <td>– 73,97</td>
      <td>40,70</td>
      <td>7,80</td>
      <td>3.75</td>
      <td>8,00</td>
      <td>2,00</td>
      <td>9,00</td>
    </tr>
    <tr>
      <th>50 %</th>
      <td>2,00</td>
      <td>1,00</td>
      <td>1,90</td>
      <td>– 73,94</td>
      <td>40,75</td>
      <td>– 73,94</td>
      <td>40,75</td>
      <td>11,30</td>
      <td>6,50</td>
      <td>15,00</td>
      <td>3.00</td>
      <td>15,00</td>
    </tr>
    <tr>
      <th>75 %</th>
      <td>2,00</td>
      <td>1,00</td>
      <td>3.60</td>
      <td>– 73,92</td>
      <td>40,80</td>
      <td>– 73,91</td>
      <td>40,79</td>
      <td>17,80</td>
      <td>9,25</td>
      <td>22,00</td>
      <td>5.00</td>
      <td>19,00</td>
    </tr>
    <tr>
      <th>bekräftat</th>
      <td>2,00</td>
      <td>9,00</td>
      <td>97,57</td>
      <td>0,00</td>
      <td>41,93</td>
      <td>0,00</td>
      <td>41,94</td>
      <td>450,00</td>
      <td>12,00</td>
      <td>30,00</td>
      <td>6,00</td>
      <td>23,00</td>
    </tr>
  </tbody>
</table>
</div>


Från sammanfattnings statistik ser du att det finns flera fält med avvikande värden eller värden som minskar modell noggrannheten. Börja med att filtrera Lat/Long-fälten så att de ligger inom gränserna för Manhattan-ytan. Detta filtrerar längre taxi resor eller resor som är avvikande i förhållande till deras förhållande till andra funktioner.

Filtrera också fältet `tripDistance` så att det är större än noll men mindre än 31 mil (haversine avståndet mellan två Lat/Long-par). Detta eliminerar långa avvikare resor som har inkonsekventa rese kostnader.

Slutligen har `totalAmount` fältet negativa värden för taxi-priserna, vilket inte är meningsfullt i den här modellen, och `passengerCount` fältet innehåller felaktiga data med de lägsta värdena noll.

Filtrera bort dessa avvikelser med hjälp av fråge funktioner och ta sedan bort de senaste kolumnerna som behövs för utbildning.


```python
final_df = green_taxi_df.query("pickupLatitude>=40.53 and pickupLatitude<=40.88")
final_df = final_df.query("pickupLongitude>=-74.09 and pickupLongitude<=-73.72")
final_df = final_df.query("tripDistance>=0.25 and tripDistance<31")
final_df = final_df.query("passengerCount>0 and totalAmount>0")

columns_to_remove_for_training = ["pickupLongitude", "pickupLatitude", "dropoffLongitude", "dropoffLatitude"]
for col in columns_to_remove_for_training:
    final_df.pop(col)
```

Anropa `describe()` igen på data för att säkerställa att rengöringen fungerade som förväntat. Nu har du en för beredd och rensad uppsättning taxi-, helgdags-och väder uppgifter som du kan använda för utbildning av maskin inlärnings modell.

```python
final_df.describe()
```

## <a name="configure-workspace"></a>Konfigurera arbetsyta

Skapa ett arbetsyteobjekt från den befintliga arbetsytan. En [arbets yta](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py) är en klass som godkänner din Azure-prenumeration och resursinformation. Den skapar också en molnresurs för att övervaka och spåra dina körningar i modellen. `Workspace.from_config()`läser filen **config. JSON** och läser in autentiseringsinformationen till ett objekt med namnet `ws`. `ws` används i resten av koden i den här självstudien.

```python
from azureml.core.workspace import Workspace
ws = Workspace.from_config()
```

## <a name="split-the-data-into-train-and-test-sets"></a>Dela data till uppsättningar för träning och testning

Dela data i utbildnings-och test uppsättningar genom att `train_test_split` använda funktionen i `scikit-learn` biblioteket. Den här funktionen åtskiljer data i data uppsättningen x (**funktioner**) för modell utbildning och data uppsättningen y (**värden att Förutsäg**) för testning.

Parametern `test_size` anger procentandelen av data som ska allokeras till testning. `random_state` Parametern anger ett start värde för den slumpmässiga generatorn, så att dina träna-test-delningar är deterministiska.

```python
from sklearn.model_selection import train_test_split

y_df = final_df.pop("totalAmount")
x_df = final_df

x_train, x_test, y_train, y_test = train_test_split(x_df, y_df, test_size=0.2, random_state=223)
```

Syftet med det här steget är att ha datapunkter för att testa den färdiga modell som inte har använts för att träna modellen, detta för att mäta den äkta noggrannheten.

Med andra ord bör en korrekt tränad modell kunna göra noggranna förutsägelser från data som den inte redan har sett. Nu har du data för automatisk utbildning av Machine Learning-modellen.

## <a name="automatically-train-a-model"></a>Träna en modell automatiskt

För att träna en modell automatiskt gör du följande:
1. Definiera inställningar för körningen av experimentet. Koppla dina träningsdata till konfigurationen och ändra de inställningar som styr träningsprocessen.
1. Skicka experimentet för modelljustering. När experimentet har skickats itererar processen genom olika maskininlärningsalgoritmer och inställningar för hyperparametrar enligt dina definierade begränsningar. Den väljer den modell som passar bäst genom att optimera ett noggrannhetsmått.

### <a name="define-training-settings"></a>Definiera utbildnings inställningar

Definiera experiment parametern och modell inställningarna för träning. Visa hela listan med [inställningar](how-to-configure-auto-train.md). Att skicka experimentet med dessa standardinställningar tar cirka 5-20 min, men om du vill ha en kortare körnings tid minskar du `experiment_timeout_minutes` parametern.

|Egenskap| Värde i den här självstudien |Beskrivning|
|----|----|---|
|**iteration_timeout_minutes**|2|Tidsgräns i minuter för varje iteration. Minska det här värdet om du vill minska den totala körningstiden.|
|**experiment_timeout_minutes**|20|Den maximala tid i minuter som alla iterationer kombineras kan ta innan experimentet avslutas.|
|**enable_early_stopping**|Sant|Flagga för att aktivera tidig uppsägning om poängen inte förbättras på kort sikt.|
|**primary_metric**| spearman_correlation | Mått som du vill optimera. Den modell som passar bäst väljs utifrån det här måttet.|
|**funktionalisering**| disk | Genom att använda **Auto**kan experimentet Förbearbeta indata (hantering av saknade data, konvertera text till numeriskt osv.)|
|**verbosity**| logging.INFO | Styr loggningsnivån.|
|**n_cross_validations**|5|Det antal delningar av korsvalidering som ska utföras när verifieringsdata inte har angetts.|

```python
import logging

automl_settings = {
    "iteration_timeout_minutes": 2,
    "experiment_timeout_minutes": 20,
    "enable_early_stopping": True,
    "primary_metric": 'spearman_correlation',
    "featurization": 'auto',
    "verbosity": logging.INFO,
    "n_cross_validations": 5
}
```

Använd dina definierade utbildnings inställningar som en `**kwargs` parameter till ett `AutoMLConfig` objekt. Ange även dina träningsdata och modelltypen, som i det här fallet är `regression`.

```python
from azureml.train.automl import AutoMLConfig

automl_config = AutoMLConfig(task='regression',
                             debug_log='automated_ml_errors.log',
                             X=x_train.values,
                             y=y_train.values.flatten(),
                             **automl_settings)
```

> [!NOTE]
> Automatiserad bearbetning av Machine Learning för bearbetning (funktions normalisering, hantering av saknade data, konvertering av text till tal osv.) blir en del av den underliggande modellen. När du använder modellen för förutsägelser tillämpas samma för bearbetnings steg som tillämpas på dina indata-data automatiskt.

### <a name="train-the-automatic-regression-model"></a>Träna den automatiska regressionsmodellen

Skapa ett experiment objekt i din arbets yta. Ett experiment fungerar som en behållare för dina enskilda körningar. Skicka det definierade `automl_config` objektet till experimentet och ange att utdata `True` ska se förloppet under körningen.

När experimentet har startats visas uppdateringarna Live som experimentet. För varje iteration ser du modelltypen, körningens varaktighet samt träningens noggrannhet. Fältet `BEST` spårar den bästa löpande körningspoängen utifrån din måttyp.

```python
from azureml.core.experiment import Experiment
experiment = Experiment(ws, "taxi-experiment")
local_run = experiment.submit(automl_config, show_output=True)
```

    Running on local machine
    Parent Run ID: AutoML_1766cdf7-56cf-4b28-a340-c4aeee15b12b
    Current status: DatasetFeaturization. Beginning to featurize the dataset.
    Current status: DatasetEvaluation. Gathering dataset statistics.
    Current status: FeaturesGeneration. Generating features for the dataset.
    Current status: DatasetFeaturizationCompleted. Completed featurizing the dataset.
    Current status: DatasetCrossValidationSplit. Generating individually featurized CV splits.
    Current status: ModelSelection. Beginning model selection.

    ****************************************************************************************************
    ITERATION: The iteration being evaluated.
    PIPELINE: A summary description of the pipeline being evaluated.
    DURATION: Time taken for the current iteration.
    METRIC: The result of computing score on the fitted pipeline.
    BEST: The best observed score thus far.
    ****************************************************************************************************

     ITERATION   PIPELINE                                       DURATION      METRIC      BEST
             0   StandardScalerWrapper RandomForest             0:00:16       0.8746    0.8746
             1   MinMaxScaler RandomForest                      0:00:15       0.9468    0.9468
             2   StandardScalerWrapper ExtremeRandomTrees       0:00:09       0.9303    0.9468
             3   StandardScalerWrapper LightGBM                 0:00:10       0.9424    0.9468
             4   RobustScaler DecisionTree                      0:00:09       0.9449    0.9468
             5   StandardScalerWrapper LassoLars                0:00:09       0.9440    0.9468
             6   StandardScalerWrapper LightGBM                 0:00:10       0.9282    0.9468
             7   StandardScalerWrapper RandomForest             0:00:12       0.8946    0.9468
             8   StandardScalerWrapper LassoLars                0:00:16       0.9439    0.9468
             9   MinMaxScaler ExtremeRandomTrees                0:00:35       0.9199    0.9468
            10   RobustScaler ExtremeRandomTrees                0:00:19       0.9411    0.9468
            11   StandardScalerWrapper ExtremeRandomTrees       0:00:13       0.9077    0.9468
            12   StandardScalerWrapper LassoLars                0:00:15       0.9433    0.9468
            13   MinMaxScaler ExtremeRandomTrees                0:00:14       0.9186    0.9468
            14   RobustScaler RandomForest                      0:00:10       0.8810    0.9468
            15   StandardScalerWrapper LassoLars                0:00:55       0.9433    0.9468
            16   StandardScalerWrapper ExtremeRandomTrees       0:00:13       0.9026    0.9468
            17   StandardScalerWrapper RandomForest             0:00:13       0.9140    0.9468
            18   VotingEnsemble                                 0:00:23       0.9471    0.9471
            19   StackEnsemble                                  0:00:27       0.9463    0.9471

## <a name="explore-the-results"></a>Utforska resultaten

Utforska resultatet av automatisk utbildning med en [Jupyter-widget](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets?view=azure-ml-py). Widgeten gör att du kan se en graf och tabell över alla enskilda körnings iterationer, tillsammans med utbildnings precisions mått och metadata. Dessutom kan du filtrera efter olika precisions mått än ditt primära mått med list rutan.

```python
from azureml.widgets import RunDetails
RunDetails(local_run).show()
```

![Jupyter Widget-körningsinformation](./media/tutorial-auto-train-models/automl-dash-output.png)
![Jupyter Widget-diagram](./media/tutorial-auto-train-models/automl-chart-output.png)

### <a name="retrieve-the-best-model"></a>Hämta den bästa modellen

Välj den bästa modellen från dina iterationer. `get_output` Funktionen returnerar den bästa körningen och den monterade modellen för den senaste pass-anropet. Genom att använda överlagringarna kan `get_output`du hämta den bästa körnings-och den monterade modellen för alla inloggade mått eller en viss iteration.

```python
best_run, fitted_model = local_run.get_output()
print(best_run)
print(fitted_model)
```

### <a name="test-the-best-model-accuracy"></a>Testa den bästa modellens precision

Använd den bästa modellen för att köra förutsägelser på test data uppsättningen för att förutse taxi-priser. Funktionen `predict` använder den bästa modellen och förutsäger värdena för y, **rese kostnad**, från `x_test` data uppsättningen. Skriv ut de 10 första förutsagda kostnadsvärdena från `y_predict`.

```python
y_predict = fitted_model.predict(x_test.values)
print(y_predict[:10])
```

Beräkna `root mean squared error` för resultatet. Konvertera `y_test` dataframe till en lista som ska jämföras med de förväntade värdena. Funktionen `mean_squared_error` tar emot två matriser med värden och beräknar det genomsnittliga kvadratfelet mellan dem. Att ta kvadratroten ur resultatet ger ett fel i samma enheter som y-variabeln, **kostnad**. Det anger ungefär hur långt priset för taxi avgiften kommer från de faktiska priserna.

```python
from sklearn.metrics import mean_squared_error
from math import sqrt

y_actual = y_test.values.flatten().tolist()
rmse = sqrt(mean_squared_error(y_actual, y_predict))
rmse
```

Kör följande kod för att beräkna medelvärde för absoluta procent fel (MAPE) med hjälp av `y_actual` fullständig `y_predict` och data uppsättningar. Det här måttet beräknar en absolut skillnad mellan varje förväntat och faktiskt värde och summerar alla skillnaderna. Sedan uttrycker den summan som en procent andel av summan av de faktiska värdena.

```python
sum_actuals = sum_errors = 0

for actual_val, predict_val in zip(y_actual, y_predict):
    abs_error = actual_val - predict_val
    if abs_error < 0:
        abs_error = abs_error * -1

    sum_errors = sum_errors + abs_error
    sum_actuals = sum_actuals + actual_val

mean_abs_percent_error = sum_errors / sum_actuals
print("Model MAPE:")
print(mean_abs_percent_error)
print()
print("Model Accuracy:")
print(1 - mean_abs_percent_error)
```

    Model MAPE:
    0.14353867606052823

    Model Accuracy:
    0.8564613239394718


Från de två noggrannhets måtten för förutsägelse ser du att modellen är ganska lämplig vid förutsägelse av taxi biljett från data uppsättningens funktioner, vanligt vis inom +-$4,00 och cirka 15% fel.

Utvecklingsprocessen för maskininlärningsmodeller är mycket resurskrävande och fordrar betydande domänkunskap och tid för att köra och jämföra resultat från flera olika modeller. Användning av automatisk maskininlärning är ett bra sätt att snabbt testa flera olika modeller för ett scenario.

## <a name="clean-up-resources"></a>Rensa resurser

Slutför inte det här avsnittet om du tänker köra andra Azure Machine Learning själv studie kurser.

### <a name="stop-the-compute-instance"></a>Stoppa beräknings instansen

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Ta bort allt

Om du inte planerar att använda de resurser som du har skapat tar du bort dem, så du debiteras inte några avgifter.

1. I Azure-portalen väljer du **Resursgrupper** längst till vänster.
1. Välj den resursgrupp i listan som du har skapat.
1. Välj **Ta bort resursgrupp**.
1. Ange resursgruppsnamnet. Välj sedan **Ta bort**.

Du kan också behålla resursgruppen men ta bort en enstaka arbetsyta. Visa arbetsytans egenskaper och välj **Ta bort**.

## <a name="next-steps"></a>Nästa steg

I den här automatiserade självstudiekursen om maskininlärning har du gjort följande uppgifter:

> [!div class="checklist"]
> * Konfigurerat en arbetsyta och förberett data för ett experiment.
> * Tränat med hjälp av en automatiserad regressionsmodell lokalt med anpassade parametrar.
> * Utforskat och granskat träningsresultat.

[Distribuera modellen](tutorial-deploy-models-with-aml.md) med Azure Machine Learning.
