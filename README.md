# notebooks

## msticpy_ti_providers

Adds additional TI providers to msticpy.

Place this notebook in the same folder as your analytics notebook. To be able reference the classes and functions add the following code to your notebook and execute it.

```python
#Install ipynb into your kernel (this has to be done once)
!pip install ipynb

# Import the functions and classes from the msticpy_ti_providers notebook
from ipynb.fs.full.msticpy_ti_providers import Shodan, load_ti_providers
```

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
The Shodan provider needs the following config paramters:

* shodan_api_key: this is your API from your Shodan subscription.
* shodan_sleep_time: the time in seconds between each request to Shodan.

The shodan_sleep_time parameter is required because Shodan requires the requests to have a 500ms interval. If too many requests in a short period are being send to Shodan it will throw an HTTP error.
If you are using the TI provider to execute single manual requests, you can set this value to 0. If it's being used to enrich data frames use a value somewhere around 0.3 to 0.5 seconds (this may require some tuning, to be safe use 0.5).
