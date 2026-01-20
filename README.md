# *Klebsiella* Genome Reports
Configuration and template files for BIGSdb Reports plugin for the Pasteur
*Klebsiella* database.

To enable these reports on the Pasteur *Klebsiella* database, you need to do the following:

* Install Weasyprint - this is needed to convert HTML documents to PDFs.
* Enable the plugin in the database configuration.
* Copy the configuration and templates to the database config directory.
* Tweak the templates to your needs.
* Ensure Kleborate has run.

## Install WeasyPrint
On Ubuntu systems this can be installed with:

```
sudo apt install weasyprint
```

Then add the path to this in /etc/bigsdb.conf by adding the following line:

```
weasyprint_path=/usr/bin/weasyprint
```

## Enable the Reports plugin
The Reports plugin is disabled by default, even in database configurations with
all_plugins="yes" set. You need to enable it by adding the following to the
database config.xml file

```
Reports="yes"
```

## Copy the configuration and templates to the database config directory
Copy the Reports directory to the database configuration directory, i.e. you 
should have a directory called 
/etc/bigsdb/dbases/pubmlst_klebsiella_isolates/Reports that contains a TOML
file describing each of the reports and three template files.

## Tweak the templates to your needs
The TOML file `reports.toml` registers the templates with the Reports plugin.
The plugin extracts all the information about an isolate from the database and
then passes this to a templating system. All isolate table fields, sequence bin
stats, assembly checks, and allelic designations for all loci are automatically
included. Scheme information, e.g. ST, cgST and other scheme fields have to be
specifically registered, as do LINcode results since gathering this 
information requires multiple database calls and if the results for a 
particular scheme are not used in a template then this is a waste of time. The
TOML file is structured as below. In templates 2 and 3 we have registered 
scheme 1 as we require MLST results, and for LINcodes we have registered scheme
18 (the cgMLST scheme) and say that we want to include isolates in the report
that have the same first 5 digit LINcode prefix. In the report we want to be
able to include isolate, country and isolation_year for these records.

```
reports = [
  {
		name		= 'Level 1',
		template_file 	= 'clinical.tt',
		comments        = 'Clinical / microbiology use',
		requires        = {
				schemes = [1]		 
		}
  },
	{
		name		= 'Level 2',
		template_file 	= 'public_health.tt',
		comments        = 'Public health use',
		requires        = {
				schemes = [1],
				lincodes = {
					 scheme = 18,
					 clusters = {
					 	 thresholds = [5], #One-based index
						 fields = ['isolate','country','isolation_year']
					 }
					 
				}
		}

	},
		{
		name		= 'Level 3',
		template_file 	= 'genomic.tt',
		comments        = 'Genomic epidemiologist use',
		requires        = {
				schemes = [1],
				lincodes = {
					 scheme = 18,
					 clusters = {
					 	 thresholds = [5], #One-based index
						 fields = ['isolate','country','isolation_year']
					 }
					 
				}
		}

	},
]
```

The template files are written for 
[Template Toolkit](http://template-toolkit.org/), which combines HTML with
processing logic.

## Ensure Kleborate and PlasmidFinder have run
Results from Kleborate are needed for the reports. These are stored for an 
isolate record when the Kleborate plugin is run.
To ensure that Kleborate results are available for all records, you should run
it using the `update_kleborate.pl` script found in the BIGSdb 
scripts/maintenance directory.

Results are also needed from PlasmidFinder. As with Kleborate, these are stored
for an isolate record when the PlasmidFinder plugin is run. Run the
`update_plasmidfinder.pl` script found in the BIGSdb scripts/maintenance 
directory to run this for all records in the database.
