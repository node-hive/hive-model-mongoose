Hive-model-mongoose
===================

Hive-model-mongoose is a superclass for mongoose model. It exists for several purposes:

1) to create a more REST-like API for the mongoose model methods
2) to insulate all of the mongoose model methods from your particular model class to reduce namespace collision.
3) to bundle a set of features with all models, including archiving and soft deletion

Note that in hive-model-mongoose, the assumption is that the models functions, not the records' native functions,
are called.

Added Functionality
-------------------

All models have "soft deletion" built in to the delete method. if Model.delete(record, callback, true) is called,
the models' delete flag is set to true, but the record itself is preserved.

Also, you can archive your data -- clone its properties into an _archive array - if you include the property
_archive ['mixed'] in your schema. This is a handy way to "back up" your data before updating it.

API
---

Hive_Model_Mongoose is a hive-component (https://github.com/bingomanatee/hive-component). Instances have the mongoose
Model in their `model` property.

### Constructor

Mongoose_Model is a factory that returns the mongoose_model via callback (and directly).

``` javascript

Mongoose_Model(
    {
        name: 'tribbles'
    } // mixins
    , {
        mongoose:   mongoose,
        schema_def: object || mongoose.Schema || path_to.jso
    } // configurations
    , dataspace // hive-model.Dataspace || Object (optional)
    , callback (optional)
    );

```

#### Mixins {object}

Any methods you want to attach to the model go here. The only requirement is the name property,
which must be a unique string (unique to your database/the dataspace)

#### Configuration {Object}

as a hive-component, a hive-model has a configuration registry that can be accessed with
* model.get_config(key):value,
* model.set_config(key, value)
* model.has_config(key):boolean

There are two required configurations:
1. mongoose -- an instance of your Mongoose module
2. schema_def -- either a Mongoose Schema, an object, or a path to a JSON file.

#### Dataspace {hive-model.Dataspace || Objet}

Dataspace can be a formal model registry (as part of the [hive-model](https://github.com/bingomanatee/hive-model) system)
or a basic object that you want to use to collect your models by name. You can pass a naked object {} in as well.

#### Callback function(err, my_mongoose_model)

a function that returns the model.

### add([records] {array}, callback{function}, as_group{boolean});

Adds a series of objects to the collection. Can be raw objects, Documents, or a combination.
If records is not an array, it routes to put(..).
If as_group != true, it feeds records one at a time to put and returns the aggregate results to the callback.
If it is true, it uses this.model.collection.insert -- meaning no property validation is done,
 and the callback will not receive the new documents.

### put ( doc {mongoose.Document || Object}, options {object}, callback {function}) || (doc, callback)
### post " "

Inserts or updates a single document.

### get (id, fields, options, callback)

Gets a single document, by ID; a passthrough to findById.

### revise(data {object}, callback{function})
update a single document's selected fields. data must have an _id property.
The other fields update the content of the object on a field by field basis.

*  if your data has a __REMOVE property, whose values are an array of strings, those properties are removed
from the document. However if the property is an array, it is emptied rather than deleted.

### all(callback {function} [optional])

returns the entire collection in the callback, or a query.

### active(callback {function} [optional])

returns the records collection in the callback, or a query. Respects soft deletes.

### inactive(callback {function} [optional])

returns the soft-deleted records in the collection in the callback, or a query.


### find (crit, field, options, callback)

passthrough to Mongoose Model.find

### find_one (crit, field, options, callback)

passthrough to Mongoose Model.findOne

### count()

passthrough to Mongoose Model.count

### empty()

drops the entire collection

### Mongoose_Model.factory(dataspace)

A static method of the Mongoose_Model.

returns a function with the API function (mixins, config, callback) that you can call
to make mongoose models without passing in the dataspace parameter all the time.