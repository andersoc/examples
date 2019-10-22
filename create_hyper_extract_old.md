
  # <center>How to create a Hyper file with new Hyper API </center>
<img src="https://pbs.twimg.com/card_img/1179423456900915201/1wErfOFR?format=jpg&name=240x240"/>

<img src="https://cdns.tblsft.com/sites/default/files/blog/picture1_87.png"/>

Whats is [**Hyper API**](https://www.tableau.com/about/blog/2019/10/deliver-results-hyper-speed)?

Hyper API is evolution of our Extract API.

What is the new features in the **Hyper API**?

1. Full CRUD: Read, update, delete, and insert data in .hyper files.
2. Full speed: Leverage the full speed of Hyper for creating .hyper files.
3. Direct CSV loading: Directly load from .CSV files instead of writing code to do so.
4. SQL-based API: Unleash the power of SQL to interact with .hyper files.
5. Multi-table: Create multi-table extracts that match your data model.

**Use Cases:** 
Connect to data sources with the Hyper API and write the data into extract files (in the .hyper file format for Tableau 10.5 and later). Write custom scripts that update data in existing extract files or read data from them.

**Benefits:** 
If you can connect to your data, you can use the Hyper API to create data extracts that improve performance and provide offline access. If you have data sources that are not currently supported, you can use the Hyper API to get the data into Tableau. If you want to update data within extract files, you can use the Hyper API to update the extract. If you need to access data from an extract, you can now write a script that reads the data from the extract.

#### As example, I am going to read the csv autos. This csv has 371527 records. To simplify this axemple, I am going to create a new dataframe with 3 columns.


```python
import pandas as pd
autos = pd.read_csv("/home/anderson/dataquest/datasources/autos.csv", encoding='Latin-1')
autos.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 371528 entries, 0 to 371527
    Data columns (total 20 columns):
    dateCrawled            371528 non-null object
    name                   371528 non-null object
    seller                 371528 non-null object
    offerType              371528 non-null object
    price                  371528 non-null int64
    abtest                 371528 non-null object
    vehicleType            333659 non-null object
    yearOfRegistration     371528 non-null int64
    gearbox                351319 non-null object
    powerPS                371528 non-null int64
    model                  351044 non-null object
    kilometer              371528 non-null int64
    monthOfRegistration    371528 non-null int64
    fuelType               338142 non-null object
    brand                  371528 non-null object
    notRepairedDamage      299468 non-null object
    dateCreated            371528 non-null object
    nrOfPictures           371528 non-null int64
    postalCode             371528 non-null int64
    lastSeen               371528 non-null object
    dtypes: int64(7), object(13)
    memory usage: 56.7+ MB



```python
auto_cp = autos[['name', 'seller','price' ]]
```


```python
auto_cp.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

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
      <th>name</th>
      <th>seller</th>
      <th>price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Golf_3_1.6</td>
      <td>privat</td>
      <td>480</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A5_Sportback_2.7_Tdi</td>
      <td>privat</td>
      <td>18300</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jeep_Grand_Cherokee_"Overland"</td>
      <td>privat</td>
      <td>9800</td>
    </tr>
    <tr>
      <th>3</th>
      <td>GOLF_4_1_4__3TÜRER</td>
      <td>privat</td>
      <td>1500</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Skoda_Fabia_1.4_TDI_PD_Classic</td>
      <td>privat</td>
      <td>3600</td>
    </tr>
  </tbody>
</table>
</div>



#### The dataframe was reduced to 3 columns


```python
auto_cp.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 371528 entries, 0 to 371527
    Data columns (total 3 columns):
    name      371528 non-null object
    seller    371528 non-null object
    price     371528 non-null int64
    dtypes: int64(1), object(2)
    memory usage: 8.5+ MB


#### Import the Hyper API Library


```python
from tableauhyperapi import HyperProcess, Telemetry, Connection, CreateMode, NOT_NULLABLE, NULLABLE, SqlType,TableDefinition, Inserter, escape_name, escape_string_literal, HyperException, TableName, HyperException
```

#### The HyperProcess is going to start up a Hyper Database Server local. 


```python
from IPython.display import Image
Image(filename="/mnt/c/Users/ander/Downloads/annotation.png")
```




![png](output_10_0.png)




```python
HyperProcess = HyperProcess(telemetry=Telemetry.DO_NOT_SEND_USAGE_DATA_TO_TABLEAU)
```

#### After that, I am going to create the name of the file and the create mode.


```python
connection = Connection(HyperProcess.endpoint, 'new_teste_addrowsxxxxx.hyper', CreateMode.CREATE_AND_REPLACE)
```


```python
connection.catalog.create_schema('Extract')
```

#### For more informations about the data types supported in this API consult the link bellow:

[https://help.tableau.com/current/api/hyper_api/en-us/reference/sql/datatype.html]('https://help.tableau.com/current/api/hyper_api/en-us/reference/sql/datatype.html')



```python
schema = TableDefinition( TableName('Extract','Extract'), [
    TableDefinition.Column('name', SqlType.text()),
    TableDefinition.Column('seller', SqlType.text()),
    TableDefinition.Column('price', SqlType.double()),    
])
```


```python
connection.catalog.create_table(schema)
```


```python
inserter = Inserter(connection, schema)
```


```python
for index, row in auto_cp.iterrows():
    inserter.add_row([row['name'], row['seller'],row['price']])      
```


```python
inserter.execute()
```

#### It is very important to do a shutdown in the HyperProcess. 


```python
connection.close()
inserter.close()
HyperProcess.shutdown()

print('The Extract Load Ended')

```

    The Extract Load Ended



```python

```
