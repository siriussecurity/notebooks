# notebooks

## Referencing the notebooks

Place these notebooks in the same folder as your analytics notebook. To be able reference the classes and functions add the following code to your notebook and execute it.

```python
#Install ipynb into your kernel (this has to be done once)
!pip install ipynb

# Import the functions and classes from the msticpy_ti_providers notebook
from ipynb.fs.full.msticpy_ti_providers import *
from ipynb.fs.full.msticpy_data_drivers import *
```

## msticpy_ti_providers

Adds additional TI providers to msticpy.

Unfortunately it's not possible to configure the new TI providers in the msticpyconfig.yaml, so we need an alternative method to feed the config settings to the providers.
To load the configured standard msticpy and additional providers run the following code:

```python
# Set the configuration settings for the additional TI providers
provider_configs = {"Shodan": {"shodan_api_key": [place_your_shodan_api_key_here], "shodan_sleep_time": 0.4}}

# Load all the TI providers
ti_lookup = load_ti_providers(provider_configs)
```
Currently the msticpy_ti_providers notebook contains only the Shodan provider, but a couple more will follow soon. 

### Shodan
The Shodan provider supports ipv4 observables. It will retrieve detailed information about the IP address and open ports from the Shodan API.

The Shodan provider needs the following config parameters:

* shodan_api_key: this is your API key from your Shodan subscription.
* shodan_sleep_time: the time in seconds between each request to Shodan.

The shodan_sleep_time parameter is required because Shodan requires the requests to have a 500ms interval. If too many requests in a short period are being send to Shodan it will throw an HTTP error.
If you are using the TI provider to execute single manual requests, you can set this value to 0. If it's being used to enrich data frames use a value somewhere around 0.3 to 0.5 seconds (this may require some tuning, to be safe use 0.5).

### Viewing results
To see the new provider in action, run the following code in your notebook:

```python
result = ti_lookup.lookup_ioc(observable="8.8.8.8", providers=["Shodan"])
df = ti_lookup.result_to_df(result)
df
```

## msticpy_data_drivers

Adds additional data drivers to msticpy.

### Defender 365 (MTP)
Authentication for the Defender 365 API is not based on the msticpy OAuth driver but on a "delegated permissions" model. The advantage of this is that no secrets need to be saved somewhere, but it's the user who interactively logs on (including MFA when configured). This is more secure and better suited for enterprise environments.

API limitations:
MTP API is limiting query results to 100.000 rows per time. It's also has a limit of 10 calls per minute, 10 minutes of running time every hour and 4 hours of running time a day. The maximal execution time of a single request is 10 minutes. Read more on:
https://docs.microsoft.com/en-gb/microsoft-365/security/mtp/api-advanced-hunting

To create a connection to MTP create a new QueryProvider instance using te new MTPDriver. Use the application and tenant id's to establish a connection.

```python
from msticpy.data.data_providers import QueryProvider

# Create a query provider for MTP
qry_prov_mtp = QueryProvider(data_environment='MTP', driver=MTPDriver())

# Log on to the workspace
qry_prov_mtp.connect(app_id=app_id, tenant_id=tenant_id)
```

To see the driver in action, run the following code in your notebook:

```python
mtp_query = '''
    DeviceProcessEvents 
    | take 100
    '''
    
qry_prov_mtp.exec_query(query=mtp_query)
```

