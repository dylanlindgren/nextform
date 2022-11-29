# NextForm
## What is NextForm?
NextForm is an application that allows you to easily create dynamic, ephemeral, UI-based forms in ServiceNow.

Previously, to create a UI-based form in ServiceNow you would either need to create a table, or create a record producer. Sometimes these methods are perfectly fine, however sometimes they can be more complex than needed (e.g. creating a form also creates ACL's, views, related lists etc), or not allow you to do everything you need (e.g. for a record producer, you can't save and return to it later).

NextForm is built into an easy to use Controller, allowing you to easily attach it to and configure a Form component. Simply add the Form component to your UI Builder page and you'll be offered to connect it to a NextForm with the preset shown below.

![NextForm preset UI](images/form-component-preset.png)

In the NextForm SysID insert the SysID of a record in the NextForms [`x_snc_nf_nextform`] table. This record points at a Generator [`x_snc_nf_generator`], and a Processor [`x_snc_nf_processor`].

## Generators
Generators create forms.

Example generator script:

```
(function generateNextFormObject() {

	return new x_snc_nf.NextForm()
		.addScreen()
		.addRow().addColumn()
		.addField(new x_snc_nf.NextField("string", "What is your given name?", "name_given"))
		.addRow().addColumn()
		.addField(new x_snc_nf.NextField("integer", "What is your family name?", "name_family"))
		.addField(new x_snc_nf.NextField("choice", "What is your age?", "age", {
            choices: [{
                displayValue: "Under 18",
                value: "under_18"
            },{
                displayValue: "18 and Over",
                value: "over_18"
            }],
            value: 'under_18',
            displayValue: 'Under 18'
        }))
		.addColumn()
		.addField(new x_snc_nf.NextField("dateTime", "When did you last sleep?", "sleep"))
		.addField(new x_snc_nf.NextField("dateTime", "When did you last wake up?", "wake"))
		.addScreen()
		.addRow().addColumn()
		.addField(new x_snc_nf.NextField("textarea", "Tell me about yourself?", "details", {
            maxLength: 4000
        }))
		.addScreen()
		.addRow().addColumn()
		.addField(new x_snc_nf.NextField("boolean", "Would you like to receive marketing communications?", "marketing_allowed"))
        .toJSON();

})();
```

## Processors
Processors process the user response for a form.

Example processor script:

```
(function processNextFormResult (nfData) {
	
	gs.info('NextForm Processed! ' + JSON.stringify(nfData));

})(nfData);
```