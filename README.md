# Open Astronomy Catalog Schema v1.0

This document describes the standard Open Astronomy Catalog (OAC) JSON schema.

Each object should be contained with a single JSON file that contains a single object bearing the object's primary name, which when empty represents the minimum readable entry file:

```JSON
{
	"SN1987A":{}
}
```

To comply with the standard, the object should contain a `schema` key, where `schema` is a permanent URL to the version of this document defining the file's schema (with `commithash` replaced appropriately), and a `name` key, where the `name` key should be identical to the object field name:

```JSON
{
	"SN1987A":{
		"schema":"https://github.com/astrocatalogs/schema/README.md",
		"name":"SN1987A"
	}
}
```

As JSON is a serialized format, *field order does not matter*, but it is recommended to organize the data in the output JSON files to make them more readable (for instance we sort photometry and spectra within each file by date, the data quantity fields by name, etc.), and to make the outputs more friendly to version control.

Sources are extremely important in the OAC schema, and every single piece of data added to an event JSON file **must have a source attribution**. Published data sources are preferred over secondary sources (aggregators would fall into a secondary source category), but if the data was collected by a secondary source intermediate to being collected by the creator of the file, it is best practice to also include these sources in the source list.

To maintain precision across JSON readers, it is recommended that numerical values should be stored as *strings*, rather than as floats or integers, even if the field type listed below is float/integer. This enables the user to maintain precision on their imported valued with packages such as `Decimal`, instead of potentially losing precision upon loading the JSON.

Sources of data contain several fields:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | ---
| `name` | Source name, e.g. "Catchpole et al. 1989" | String | no
| `alias` | ID number unique to this source to be used as an alias | Integer | no
| `url` | Web address of source | String | yes
| `bibcode` | 19 character NASA ADS bibcode | String | yes
| `doi` | Digital object identifier | String | yes
| `arxivid` | ArXiv ID number | String | yes
| `secondary` | Did this source collect the data from another source (i.e. second-hand source) | Boolean | yes
| `acknowledgment` | Acknowledgment requested by source if data is used in publication | String | yes

The sources object contains an array of such objects:

```JSON
"sources":[
	{
		"name":"Catchpole et al. (1987)",
		"bibcode":"1987MNRAS.229P..15C",
		"alias":"1"
	},
	{
		"name":"SUSPECT",
		"url":"https://www.nhn.ou.edu/~suspect/",
		"alias":"2",
		"secondary":true
	}
]
```

The OAC schema stores many different pieces of metadata for each event. Data quantities are added to each event as arrays of objects, with each piece of datum being tagged with its associated sources' alias tags. As an example a redshift object array may look like:

```JSON
"redshift":[
	{
		"value":"0.045",
		"e_value":"0.001",
		"source":"1,2",
		"kind":"heliocentric"
	},
	{
		"value":"0.043",
		"e_value":"0.002",
		"source":"3",
		"kind":"host"
	}
]
```

where in this example we have two different redshift values quoted from three different sources, where two of the sources agree with one another, and the third source actually refers to the redshift of the host galaxy rather than the supernova. Note that all the numerical quantities are stored as strings instead of as numbers, the OAC's policy is to store the data with exactly the same number of significant digits as the sources that provide them, and storing the importing the data as floating point numbers can often introduce small floating-point errors that we wish to avoid.

In general, the data stored in a given JSON file are presented in the observer frame, uncorrected for extinction or redshift, with times provided in MJD or as a calendar date (YYYY/MM/DD). In some instances, only corrected data will be available, or will be available as a supplement to the raw data, but in these cases the corrected data should be tagged with additional qualifiers (e.g. `kcorrected`, `deredshifted`, etc.) to indicate that it has been manipulated. As with all tags, the source material is often unclear in how they provided the data; we attempt to figure this out from the original publication when possible, but this cannot always be determined.

Data quantities have their own set of standard fields:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `value` | The value of the quantity | String, Float, or Array | no
| `e_value` | The error associated with the value | Float | yes
| `e_lower_value` | The lower error bound associated with the value | Float | yes
| `e_upper_value` | The upper error bound associated with the value | Float | yes
| `lowerlimit` | Value is a lower limit | Boolean | yes
| `upperlimit` | Value is an upper limit | Boolean | yes
| `u_value` | The unit of the value | String | yes
| `u_e_value` | The unit of the error in the value (if different from `u_value`) | String | yes
| `correlation` | Correlation of value with another value (structure described below) | Object | yes
| `kind` | Variant of the quantity | String | yes
| `derived` | Quantity is derived from other data present in file | Boolean | yes
| `description` | A description of the quantity | String | yes
| `probability` | Probability of the given value being true | Float | yes
| `source` | Comma-delimited list of integer aliases to sources for the data | String | no
| `model` | Comma-delimited list of integer aliases of which models the data originated from | String | yes
| `realization` | A numeric ID for the realization of the denoted model (e.g. from Monte Carlo) | Integer | yes

Correlations between quantities can be specified with the correlation object, which has the following structre:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `quantity` | Name of quantity this quantity is correlated with | String | no
| `value` | The value of the correlation | String or Float | no
| `kind` | The kind of correlation presented (e.g. `"Pearson"`) | String | yes
| `derived` | Correlation is derived from other data present in file | Boolean | yes

Currently, the OAC explicitly tracks the following quantities for each entry, if available (items marked with a 🌟 are transient-specific):

| Field | Description
| :--- | :---
| `alias` | Other names this event goes by
| `distinctfrom` | Names of events that this event is *not* associated with, usually very nearby events that may be confused with the given event
| `error` | Known errors in sources of data that are ignored on import
| `ra` | Right ascension of event in hours (`hh:mm:ss`)
| `dec` | Declination of event in degrees
| `discoverdate` | Date that the event was first observed
| `maxdate` | Date of the event's maximum light
| `maxabsmag` | Maximum absolute magnitude
| `maxappmag` | Maximum apparent magnitude
| `maxband` | Band that maximum was determined from
| `maxvisualabsmag` | Maximum visual absolute magnitude 🌟
| `maxvisualappmag` | Maximum visual apparent magnitude 🌟
| `maxvisualband` | Band of maximum visual magnitude 🌟
| `maxvisualdate` | Date of maximum visual magnitude 🌟
| `redshift` | Redshift of event or its host in various frames (`heliocentric`, `cmb`, `host`)
| `lumdist` | Luminosity distance to the event
| `comovingdist` | Comoving distance to the event
| `velocity` | Recessional velocity of event (`heliocentric`, `cmb`, `host`)
| `claimedtype` | Claimed type of the event 🌟
| `discoverer` | Person(s)/survey(s) who discovered the event
| `ebv` | Reddening originating from the Milky Way
| `host` | Host galaxy of the event
| `hostra` | Right ascension of the host galaxy in hours (`hh:mm:ss`)
| `hostdec` | Declination of the host galaxy in degrees
| `hostoffsetang` | Offset angle between host and event
| `hostoffsetdist` | Offset angular diameter distance between host and event

Photometry and spectra are stored in a similar way, but have different and many more standard field names. Both photometry and spectra share a few fields:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `time` | Time of observation (can be a two-element array for start/stop) | Float or Array | yes
| `e_time` | Error in the time | Float | yes
| `e_lower_time` | Lower error in the time | Float | yes
| `e_upper_time` | Upper error in the time | Float | yes
| `u_time` | Unit for time | String | yes
| `survey` | Name of survey observations were collected by | String | yes
| `instrument` | Instrument used for observation | String | yes
| `telescope` | Telescope used for observation | String | yes
| `observatory` | Observatory used for observation | String | yes
| `observer` | Person(s) who conducted the observation | String | yes
| `reducer` | Person(s) who reduced the observation | String | yes
| `airmass` | Airmass through which spectrum was collected | Float | yes
| `host` | Observation is that of the host galaxy, not the object | Boolean | yes
| `includeshost` | Host galaxy light not subtracted from observation | Boolean | yes
| `source` | Comma-delimited list of integer aliases to sources for the data | String | no
| `model` | Comma-delimited list of integer aliases of which models the data originated from | String | yes
| `realization` | A numeric ID for the realization of the denoted model (e.g. from Monte Carlo) | Integer | yes

For all photometry, count rates (and their errors) can be specified:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `countrate` | Counts per unit time. This could be literal counts detected, or a flux normalized to a zero point | Float | yes
| `e_countrate` | Error in the count rate | Float | yes
| `e_lower_countrate` | Negative error in the count rate | Float | yes
| `e_upper_countrate` | Positive error in the count rate | Float | yes

For IR/optical/UV photometry specifically, typical field names are:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `magnitude` | Apparent magnitude | Float | no
| `e_magnitude` | Error in the magnitude | Float | yes
| `e_lower_magnitude` | Lower (i.e. more negative) error in the magnitude | Float | yes
| `e_upper_magnitude` | Upper (i.e. more positive) error in the magnitude | Float | yes
| `zeropoint` | Zero point value used to covert flux to magnitudes | Float | yes
| `band` | Photometric band filter used | String | yes
| `bandset` | Set of bands that filter is drawn from (e.g. Johnson, Swift) | String | yes
| `system` | Photometric system used | String | yes
| `upperlimit` | Measurement is an upper limit | Boolean | yes
| `upperlimitsigma` | Number of sigmas upper limit corresponds to | Float | yes
| `kcorrected` | Photometry has been K-corrected for redshift effects | Boolean | yes
| `scorrected` | Photometry has been S-corrected for extinction in host galaxy | Boolean | yes
| `mcorrected` | Photometry has been S-corrected for extinction from Milky Way | Boolean | yes

For radio, a few more field names are used:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `frequency` | Frequency of observation | Float | yes
| `u_frequency` | Unit for frequency | String | yes
| `fluxdensity` | Flux density | Float | no
| `e_fluxdensity` | Error in flux density | Float | yes
| `e_lower_fluxdensity` | Negative error in the flux density | Float | yes
| `e_upper_fluxdensity` | Positive error in the flux density | Float | yes
| `u_fluxdensity` | Unit for flux density | String | yes
| `upperlimit` | Measurement is an upper limit | Boolean | yes

For X-ray, the additional set of fields are:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `energy` | Detector energy (can be a two-element array for range) | Float or Array | yes
| `u_energy` | Unit of energy | String | yes
| `flux` | Energy flux | Float | no
| `unabsorbedflux` | Unabsorbed energy flux | Float | yes
| `photonindex` | Power-law assumed to convert counts to flux | Float | yes
| `nhmw` | Milky Way hydrogen column density | Float | yes
| `e_flux` | Error in the flux | Float | yes
| `e_lower_flux` | Negative error in the flux | Float | yes
| `e_upper_flux` | Positive error in the flux | Float | yes
| `u_flux` | Unit for flux | String | yes
| `upperlimit` | Measurement is an upper limit | Boolean | yes

And finally for spectra, these fields are used:

| Field | Description | Type | Optional?
| :--- | :--- | :--- | :---
| `data` | Array of wavelengths, fluxes, and (optionally) errors | Nx2 or Nx3 array | no
| `u_wavelengths` | Unit for wavelength | String | no
| `u_fluxes` | Unit for fluxes | String | no
| `u_errors` | Unit for flux errors | String | yes
| `snr` | Signal to noise ratio | Float | yes
| `filename` | Name of file spectra was extracted from | String | yes
| `deredshifted` | Data is known to have been deredshifted from observer frame | Boolean | yes
| `dereddened` | Data is known to have been dereddened | Boolean | yes
| `vacuumwavelengths` | Wavelengths reported are what they would be if measured in a vacuum (as opposed to air) | Boolean | yes
| `exclude` | Suggested wavelengths (in &#8491;) to exclude when plotting/analyzing, can be `above`, `below`, or `range`, e.g. `"above":"10000"` would suggested excluding data from wavelengths greater than 10,000 &#8491;, `"range":["8000","8100"]` would suggested excluding data from wavelengths in between 8,000 and 8,100 &#8491; | Object | yes
| `source` | Comma-delimited list of integer aliases to sources for the data | String | no

So long as it is reasonable, the OAC is open to adding more field names should additional information need to be stored in an event file beyond the quantities and data we have chosen to track here, please contact us and let us know if you have any suggestions on how the standard format can be improved.
