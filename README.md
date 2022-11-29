# NextForm
## What is NextForm?
NextForm is an application that allows you to easily create dynamic, ephemeral, UI-based forms in ServiceNow.

Previously, to create form displayed in the UI of ServiceNow you would either need to create a table, or create a record producer. Sometimes these methods are perfectly fine, however other times they can be more complex than needed. For example:

- If you create a table, it means you'll get all the overhead and complexity in your application that comes with such as ACL's, views, related lists, UI actions etc.
- If you create a record producer, you can only use it to create new records. They are not designed to support opening existing records and editing those values.

NextForm allows you to dynamically define a form that will only ever exist in the UI. The values for the form can be pre-populated with data sourced from the ServiceNow instance, and once completed the values can also be persisted into storage on the ServiceNow instance. However, how this generation of a form is done, and how one is processed, is completely left up to the developer leveraging NextForm.

![NextForm Overview](images/nextform-overview.png)

NextForm's power comes from its Controller; This contains all the front-end logic to drive the experience. Using its [Preset](https://docs.servicenow.com/bundle/tokyo-application-development/page/administer/ui-builder/concept/presets.html), you can easily attach it to and configure a Form component. Simply add the Form component to your UI Builder page and you'll be offered to connect it to a NextForm with the prompt shown below:

![NextForm "Preset" prompt](images/form-component-preset.png)

In the NextForm SysID insert the SysID of a record in the NextForms [`x_snc_nf_nextform`] table. This record points at a Generator [`x_snc_nf_generator`], and a Processor [`x_snc_nf_processor`].

## Generators
Generators are what creates the form that will be displayed by NextForm. They must return an `x_snc_nf.NextForm` object.

The NextForm object has a number of helper functions to allow you to define your form. Each function returns a reference to itself, so you can use method chaining to define forms in an expressive manner.

### The NextForm Generator API

#### NextForm
##### `new NextForm()`
Creates a `NextForm`. There are no parameters required.

##### `NextForm.addScreen()`
Add a screen to the NextForm. There are no parameters required.

##### `NextForm.addRow()`
A method to layout a screen. Adds a vertical row to the most recently created `NextScreen`, that columns can be placed inside. There are no parameters required.

##### `NextForm.addColumn()`
A method to layout a screen. Adds a horizontal column to the most recently created `NextRow`, that fields can be placed inside. There are no parameters required.

##### `NextForm.addField(x_snc_nf.NextField)`
Adds a field to the most recently created `NextColumn`. Accepts a `NextField` as its only parameter.

#### NextField
##### `new NextField(type, label, name, options)`
Creates a `NextField`. Accepts three mandatory parameters, and one optional:

###### type
Mandatory. The field type as a string. Current supported types are `string`, `integer`, `choice`, `dateTime`, and `boolean`. Depending on the type of field, different options may be supported.

###### label
Mandatory. The label for the field.

###### name
Mandatory. The internal name of the field. This must be unique across the entire NextForm.

###### options
Optional. An object with properties and values corresponding to the option for the field you want to set. The following options can be set:

```
{
    maxLength: 1400,
    choices: [{
        displayValue: "Option 1"
        value: "1"
    },{
        displayValue: "Option 2"
        value: "2"
    }]
}
```

### Example Generator Script

```
(function generateNextFormObject () {

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
            }, {
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
        .addField(new x_snc_nf.NextField("boolean", "Would you like to receive marketing communications?", "marketing_allowed"));

})();
```

## Processors
Processors process the user response for a form as required by the form developer.

The `nfData` object contains properties for each of the fields that were created in the Generator script. In those properties, you can access the `value` and `displayValue` that was set for those fields.

### Example Processor Script

```
(function processNextFormResult (nfData) {
	
	gs.info('NextForm Processed! ' + JSON.stringify(nfData));
	
	gs.info("The user's family name was: " + nfData.name_family.displayValue);

})(nfData);
```