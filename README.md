Ember v2.7 and Sails v0.12.4
=

Summary: frontend Ember - backend Sails
--
Ember and Sails looked like a match made in heaven. With Sails' superfast APIs and Ember's superfast models, perfect!  

Not quite.  Ember was quite picky about the data received from Sails, and Sails a little tardy on specification compliance (JSONAPI), and doesn't support PATCH out of the box ( ! ).

So, here is the setup I worked out, allowing an Ember frontend to communicate with Sails backend with MariaDB at the rear.

Resources
--

* [EmberJS](http://emberjs.com/) v2.7
* [SailsJS](http://sailsjs.org/) v0.12.4
* [sails-mysql](https://www.npmjs.com/package/sails-mysql) v0.11.5
* [sails-json-api-blueprints](https://www.npmjs.com/package/sails-json-api-blueprints) v0.12.1
* [sails-generate-models](https://www.npmjs.com/package/sails-generate-models) v1.1.0
* [JSON API](http://jsonapi.org/) v1.0
* [node](https://nodejs.org/en/) v6.5.0
* [npm](https://www.npmjs.com/) v3.10.3
* [nvm](https://github.com/creationix/nvm) v0.31.6


Hurdles and Solutions
--

* Sails does not support PATCH out of the box, only PUT; so I added an alias for PATCH in routes.js to redirect PATCHs to PUTs.

* Sails doesn't talk proper JSONAPI (nor REST) so I used a Sails blueprint to format the output to JSONAPI.

* Ember needs to use it's JSONAPIAdapter.

Specifics Summary
--

* EmberJS 2.7 in it's own folder
	* with JSONAPIAdapter enabled in `app/adapters/application.js`

		```
		export default DS.JSONAPIAdapter.extend({
		  host: 'http://localhost:1337', /// 1337 is Sails's default port
		});
		```

* SailsJS 0.12.4 in it's own folder
	* `npm install --save sails-json-api-blueprints`
	* and modifications to 
		* `config/cors.js` :
			* set _allRoutes_ to `true`
			* add `PATCH` to the _methods_ attribute

		* `config/routes.js` :
			* then we add a custom route for each Sails api like this:
			* `'PATCH /individuals/:id':  'IndividualsController.update'`
			* where `Individuals` is an existing model.




Additions (1) - Linking MySQL / MariaDB
--
In addition to setting up the above communication between Ember and Sails, I also created Sails models from a schema I had created earlier in MariaDB (essentially MySQL).

To create Sails models quickly I entered Sails project directory and ran the following:

```
 npm install --save sails-mysql
 npm install --save-dev sails-generate-models
```

_Setup `sails-mysql` adapter_

I updated the database details in `config/connections.js` to the MariaDB previously created.

```
  // someMysqlServer: { },
  someMysqlServer: {
    adapter:  'sails-mysql', // there's the adapter we npm installed
    host:     'localhost',
    user:     'user',
    password: 'password',
    database: 'idv'
  },
```

* and pointed Sails towards it by updating  `config/models.js` with:

```
  // connection: 'localDiskDb',
  connection: 'someMysqlServer', // the database name from connections.js
```

_Generate Sails Models Fast!_

Then in the Sails project folder I ran `sails-generate-models`; all my models and controllers are automatically generated using the details i added about in `connection.js`.  Here's an example output:

```
# sails-generate-models

- Writing Accounts.js...
- Writing AccountsController.js...
- Writing Headings.js...
- Writing HeadingsController.js...
- Writing Idv-log.js...
...
```

With any change to the database schema I just re-run `sails-generate-models` to keep Sails's models uptodate.


Additions (2) - Creating Ember Models
--
Creating models in Ember was simply a matter of calling ember-cli and adding stubs for expected fields from the backend.

Begin with `ember g model individuals`.

And update the model file `app/models/individuals.js` with fields:

```
import DS from 'ember-data';
import attr from 'ember-data/attr';


export default DS.Model.extend({
  'first-name':    attr( ),
  'last-name':     attr( ),
  postcode:     attr( ),
  address1:     attr( ),
  address2:     attr( ),
  role:         attr( ),
  shareholding: attr( ),
  nationality:  attr( ),
  idvstatus:    attr( ),
});
```


