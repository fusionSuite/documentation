## Introduction

This chapter describe how *rule actions* works and how to write them.

## What is rule actions scripts?

When you add a new item into FusionSuite, rules are played.  
We focus on the type of rule named *notification*.  
This kind of rule is used to interact with tools outside FusionSuite.  

There are couple examples:

 * Send a notification by mail
 * Send a notification to [Slack](https://slack.com/)
 * Create a host in [Zabbix](https://www.zabbix.com/)


## How scripts works?

Here are the steps:

* Create an item
* Play rules
* Rule criteria validated
* Action of the rule is defined to call a script
* Call the script with the item properties (defined in a type of item)
* Script executed
* Script can return a value (not mandatory)
* If return value, store it in the item property (defined in the action of rule)
* End :wink:


## What's happening when script have exception

When script have problem, the exception is catched by FusionSuite.  
It's not blocking and the end of all rules are played.  
At the end of the `POST` item, return a `HTTP 500 error` with the error message.  

An example:

```
{"status":"error","message":"Error in rule `zabbix add host`: The URL3 is required"}
```

## Structure of the script

### Folder & files

The scripts are stored in root folder of FusionSuite: `/rulesExternalScripts/`.
Each script has it's own folder, for example, for *Zabbix* add host, we have a folder `actionZabbix`.
Inside, find a minimum of 2 files:

* `actionZabbix.json`: Same name than the folder and with *.json* extension. It have the item structure for parameters to pass to the script
* `actionZabbix.php`: Same name that the folder and with *.php* extension. It's the code executed by the rule action


Here is an example of the structure
```
rulesExternalScripts
├── actionZabbix
|   ├── actionZabbix.json
|   ├── actionZabbix.php
```

### How to use my own libs?

The script can have it's hown libs, use `composer require yourlib`.  
When *composer* ask you `No composer.json in current directory, do you want to use the one at /xxx/FusionSuite/backend? [Y,n]?`, **you must say n**.

### The *PHP* script file

This is an example with *Zabbix* script:

???+ example
    ```php
    <?php

    /* Declare the namespace: RuleExternalScript followed by script folder */
    namespace RuleExternalScript\actionZabbix;

    /* include the autoload in case you have libs installed with composer */
    include __DIR__.'/vendor/autoload.php';

    /* Call the lib */
    use IntelliTrend\Zabbix\ZabbixApi;

    /* the class name: same name than the folder */
    class actionZabbix
    {
      /* the send function with arguments, it's this function called by the rule action */
      static function send($args)
      {

        /*
         * not mandatory, but highly recommended !
         * list the arguments you need and define them has required
         * if an argument is missing, the validator will throw an error
         */
        $dataFormat = [
          'URL'              => 'required',
          'username'         => 'required',
          'password'         => 'required',
          'hostname'         => 'required',
          'zabbixGoupId'     => 'required',
          'zabbixTemplateId' => 'required'
        ];
        \App\v1\Common::validateData($args, $dataFormat);

        /* the code to execute, not written here to be easier to read */
        // [...]
        /* the function of the lib to create a host in Zabbix, and get the error */
        $result = $zbx->call('host.create', $hostData);

        /* return the id of the host in Zabbix inside `value` key in an array */
        return ["value" => $result['hostids'][0]];
      }
    }
    ```

### The *JSON* script file

It's the FusionSuite item structure (name & properties).  
The goal is to create a type of item of your script with parameters to pass to it.  
In our example, we have 6 parameters to pass to our script. So we create an item with 6 properties, with same name.  
With this type of item, you can create many items. These items will be defined in rules actions.

Couple examples:

* Item named *server Linux* (with a specific Zabbix templateId) can be defined on the rule with criteria: server with operating system is Linux
* Item named *server Windows* (with a specific Zabbix templateId) can be defined on the rule with criteria: server with operating system is Windows

With this system it's powerful :wink:

???+ example "Here an example of the *JSON* file"
    ```json
    [
      {
        "itemName": "RuleAction Zabbix create host",
        "propertygroups": [
          {
            "name": "Config",
            "properties": [
              {
                "name": "url",
                "valuetype": "string",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              },
              {
                "name": "username",
                "valuetype": "string",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              },
              {
                "name": "password",
                "valuetype": "string",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              }
            ]
          },
          {
            "name": "Values",
            "properties": [
              {
                "name": "hostname",
                "valuetype": "propertyId",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              },
              {
                "name": "ip",
                "valuetype": "itemlink",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              },
              {
                "name": "groupId",
                "valuetype": "integer",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              },
              {
                "name": "templateId",
                "valuetype": "integer",
                "regexformat": "",
                "unit": "",
                "listvalues": []
              }
            ]
          }
        ]
      }
    ]

    ```

## Publish your script

### In official FusionSuite repository

You can create a Pull Request on own [backend repository](https://github.com/fusionSuite/backend).
It will be added in our CI tests (validity of code...) and will be integrated in the next release.

!!! warning
    Your script will be under GNU AFFERO GENERAL PUBLIC LICENSE, Version 3.

### In your own repository

Create a release with the folder for the users.
