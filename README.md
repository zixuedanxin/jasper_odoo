# jasper_odoo
Jasper Reports for Oddo (No Jasper Server needed)

# Before insall this module

## Install python packages

### pyJNIus package
For ubuntu distributions, PyJNIus library can be installed through PIP installer:

```
#sudo pip install Cython
#sudo pip install pyjnius
```
### dicttoxml package
For ubuntu distributions, dicttoxml can be installed through PIP installer:
```
#sudo pip install dicttoxml
```
## Check/create environment variables
For ubuntu distributions, you can set the environment variables at /etc/environment file. Edit this file in order to set the variables every time your system run.

### JAVA_HOME
For correct use of JNI interface, environment variable JAVA_HOME should be set. Below a default value for this variable, check your Java installation first!
```
JAVA_HOME="/usr/lib/jvm/default-java"
```
### CLASSPATH
To easy distribute this module, all jar files were inserted in this repository under /java folder. So, the easier way to JNI access these files is set CLASSPATH with this folder. Bellow an example of CLASSPATH if the repository was clonned in /opt/odoo/jasper/ folder:
```
CLASSPATH="/opt/odoo/jasper/jasper_odoo/java/*"
```

## Environment variables in services (daemons)
Some distributions does not recognize the environment variables set in _/etc/environment_ file.
In this case, an easy workaround is set the environment in the daemon bash file located in _/etc/init.d/_ folder. Add the following lines in your service file to set the environment variables correctly:

```
export JAVA_HOME="/usr/lib/jvm/default-java"
export CLASSPATH="/opt/odoo/jasper/jasper_odoo/java/*"
```

## Temporary folder
When this module is in use, it creates temporary files in order to generate the reports. A temporary folder with read an write access to odoo user is needed. The insallation sugest /var/jaspertemp/ (check odoo system parameters after module installed), but you can set any other folder. Below a set of commands to create and set access:
```
#sudo mkdir /var/jaspertemp
#sudo chown odoo:odoo /var/jaspertemp
#sudo chmod u+rw /var/jaspertemp
```

# Designing a report
This documentation does not detail Jasper Studio process, only the module particularities. For further documentation about Jasper Studio, refer to: http://community.jaspersoft.com/documentation

After activate the *developer mode*, you can create reports under the action -> reports on Settings menu.
Create a new folder choosing _Jasper_ as *Report Type*. After that, new options appear on *Jasper" tab.

## Accelerate the design using a XML sample file
To generate Jasper reports without need to a connection to PostgreSQL, this module generates XML documents as source to your reports. You can create a XML sample file to accelerate the design of your report by clicking the "Get XML sample" button under model field.
- Set the model field to the model you want to create
- Under Model tab on Jasper tab, set the fields you want to use. If none is set, all fields will be exported (not a good idea!)

The XML structure is something like this:
```XML
<?xml version="1.0" ?>
<odoo>
	<model_name>
		<item>
			<field1>value_of_field1</field1>
			<field2>value_of_field2</field2>
			<many2one_field.subfield>value subfield</many2one_field.subfield>
		</item>
			...
	</model_name>
	<one2many_field>
		<item>
			<model_name>id_of_model</model_name>
			<field1>value_of_field1</field2>
			<field2>value_of_field2</field2>
		</item>
			...
	<one2many_field>
</odoo>
```
### Base fields
Base fields are exported to XML file as is. Take care with binary fields, they are exported as Base64.

### Many2one fields
Many2one fields are exported as base fields on the source with the name _field.subfield_
You can choose the subfields of your model on the fields screen.

### One2Many fields and Many2Many fields
One2Many fields and Many2Many fields are exported as an extra source. The reference to the model is created with a field with the model name on this extra source. You can use thees extra source on your subreport (check next topic)

## Uploading Jasper fields
Once you designed and tested your Jasper report with the XML sample file. You need only to upload the *jrxml* and *jasper* files to the *Design file (jrxml)* and *Compiled file (.jasper)* fields respectively on the report action and save your report. After that just click on "Add report to model" button and done! Go to your model menu, select one or several records and call your jasper report.

# Working with subreports
To work with subreports, you can use any One2Many or Many2Many field of your model as data source. Once one field of theese kind is added to the model report, it is exported as an extra datasource with the name as the field model name. A field named as the upper model is inserted to your field list to be used as query on your XPath param.

## Before use subreports
When you are developing reports with subreports the *Expression* parameter of your subreport object on main report is a path to the subreport file. Unfortunally, when it comes to odoo there are no more file system and everything is in memory. So, for the module be able to work with subreports, theese are passed as parameters to main report. Because of this feature a few additional steps are needed before upload your reports to odoo as follow:

### Main report
 - On the main report, create a param of type java.lang.Object and name him for example, _mysubreport_
 - On your subreport object, change the *Expression* field (with should have a path to the subreport) for the value: _$P{mysubreport}_. This will set the report to send a param named _mysubreport_ as the subreport content.
 - Stil on subreport object, click on *Edit Parameters* and add the following parameter: _XML_DATA_DOCUMENT = $P{XML_DATA_DOCUMENT}_. This will force the report to send the XML as source for subreport

Once you have done all this, compile and upload your jasper files (jrxml and jasper) to your report on odoo.

### Subreport(s)

I presume you have already created the paramenter on the subreport to set the ID to be used as filter on your XPath. All this structure works properly on odoo. So, once the main report was adapted to work in the module, the only concern before you upload your subreport files is the *Param* field. It should have the same name as the parameter created to handle the subreport on main report.

# Calling report from code
So far, we were able to create a report and call it from a tree view of our model. Now let's suppose you need to call your report from a button for example. In this case you can call as any other report on odoo with the difference you will need to send the data to render your report. For this case, there are two ways to send your data

## Calling report sending model ids

Suppose you need to call your report from a wizard where you can get all ids from your model trough a python code. In this case, you just need to send your ids in the context with the dictionary value *jasper_ids*. Check the sample below:

```python

	context = {}
	context['jasper_ids'] = [1,2,3]

	return {
	    'type': 'ir.actions.report.xml',
	    'report_name': 'my_jasper_report_name',
	    'context' : context
	}
```

## Calling report sending full data

This is the case in wich your data will be completely generated in your code. In this case you need to send your values in the dictionary value *jasper_data*. Your data will be directely converted to XML using dicttoxml and then sent to your jasper report in a XML document. All the model definition is ignored and only the report files are used. Check the sample below:

```python

	context = {}
	context['jasper_data'] = {} 
	context['jasper_data']['mylist'] = [{'name':'John', 'age': 17},{'name':'Mary', 'age':17}]

	return {
	    'type': 'ir.actions.report.xml',
	    'report_name': 'my_jasper_report_name',
	    'context' : context
	}
```
