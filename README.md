# Word Template Exporter #
Package for easy Word exports in Laravel on given Templates.

<p style="text-align: center;">
<a href="https://github.com/santwer/exporter"><img src="https://img.shields.io/github/commit-activity/m/santwer/exporter" alt="Commit Activity"></a>
<a href="https://packagist.org/packages/santwer/exporter"><img src="https://img.shields.io/packagist/dt/santwer/exporter" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/santwer/exporter"><img src="https://img.shields.io/packagist/v/santwer/exporter" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/santwer/exporter"><img src="https://img.shields.io/packagist/l/santwer/exporter" alt="License"></a>
</p>

## Installation

Exporter is installed via [Composer](https://getcomposer.org/).
To [add a dependency](https://getcomposer.org/doc/04-schema.md#package-links) to Exporter in your project, either

Run the following to use the latest stable version
```sh
    composer require santwer/exporter
```
or if you want the latest master version
```sh
    composer require santwer/exporter:dev-master
```

You can of course also manually edit your composer.json file
```json
{
    "require": {
       "santwer/exporter": "v0.2.*"
    }
}
```

## How to use
Add Trait *Exportable*
```php
use Santwer\Exporter\Exportable;

class User {
    use Exportable;
    ...
}
```

As default all Variables are available of the Model.
Add a Concern for special export fields.
```php
use Santwer\Exporter\Exportable;
use Santwer\Exporter\Concerns\HasTokens;
...

class User implements HasTokens {
    use Exportable;
    ...
    public function exportTokens(): array
    {
        return [
            'name'  => $this->name,
            'email' => $this->email,
        ];
    }
}
```

You can add a fixed Template for each Model.
```php
use Santwer\Exporter\Exportable;
use Santwer\Exporter\Concerns\HasTemplate;
...

class User implements HasTemplate {
    use Exportable;
    ...
    public function exportTemplate(): string
    {
        return '/template.docx';
    }
}
```

You can also define own Blocknames for the use of the Model in a template. 
```php
use Santwer\Exporter\Exportable;
...

class User implements HasTemplate {
    use Exportable;
    ...
    protected $exportBlock = 'customer';
}
```

### Basic Export

export() gives download response back.
```php
return User::where(...)
        ->template('templatefile.docx')
        ->export();
```

If you defined export template in Model.
```php
return User::where(...)
        ->export();
```


```php
return User::where(...)
        ->template('templatefile.docx')
        ->store('storage/path');
```

```php
return User::where(...)
        ->template('templatefile.docx')
        ->storeAs('storage/path', 'name.docx');
```
It's also possible to set Export-Command after Executing the query or on a Model after Find-Command
```php
return User::where(...)
        ->first()
        ->export('filename.docx', ['template' =>' templatefile.docx']);
```
```php
return User::find(1234)
        ->export('filename.docx', ['template' =>' templatefile.docx']);
```

## Export as PDF
Generally with the option format = pdf it is possible to export pdf. 
It is important that libreOffice is installed for that actions.
```php
return User::where(...)
        ->template('templatefile.docx')
        ->export(['format' => 'pdf']);
```
For short terms it is possible to call Export functions as PDF
```php
return User::where(...)
        ->exportPdf();
```
```php
return User::where(...)
        ->exportFirstPdf();
```


## Autoloading Relations

Before exporting, the Package is checking for defined Relations.
If there is no related Variable it will automatically remove unneeded relations. 
This behavior can be changed within the config. For that it is needed to set up a config File exporter.php in config/

```php
return [
    'removeRelations' => false,
]
```

Also is the Package checking for relations that are not loaded yet. It will automatically load the Relations before exporting.
Therefore, it is possible to reduce the Export-Code from 
```php
return User::with('posts')
        ->with('posts.comments')
        ->with('posts.comments.user')
        ->template('templatefile.docx')
        ->export();
```
to 
```php
return User::template('templatefile.docx')->export();
```
If the Relation is already loaded it will not be affected. 

## Variables

It is possible to set up variables which are not affected by the Model or Query.
```php
use Santwer\Exporter\Processor\GlobalVariables;
...
       GlobalVariables::setVariable('Date', now()->format('d.m.Y'));
```

```php
use Santwer\Exporter\Processor\GlobalVariables;
...
      GlobalVariables::setVariables([
          'Time' =>  now()->format('H:i'),
          'Date' =>  now()->format('Y-m-d'),
      ]);
```


## Template

The Template should be DOCX or DOC. The File will be cloned and saved in the sys_temp_folder as long it has no store option. 
For PDF exports it is needed to use LibreOffice. Therefore, the soffice command needs to be executable.

For the Templateprocessing it uses [phpoffice/phpword](https://github.com/PHPOffice/PHPWord)
More Infos you can find [here](https://phpword.readthedocs.io/en/latest/templates-processing.html)

## Template Variables/Blocks

In the template the package always looks for loops/Blocks (except for Global Variables). 
By Default the Blockname is the name of the table. It is also possible to use an own name for that. 
```php
use Santwer\Exporter\Exportable;
...

class User implements HasTemplate {
    use Exportable;
    ...
    protected $exportBlock = 'customer';
}
```

To export Customers with Name and e-mail addresses, it is needed to add the Block.
```word
{customer}
    {name}, {email}
{/customer}
```

If there is a Relation within the customer.

```php
use Santwer\Exporter\Exportable;
...

class User implements HasTemplate {
    use Exportable;
    ...
    protected $exportBlock = 'customer';
    
    public function deliveryAddress()
    {
        return $this->hasOne(Address::class);
    }

}
```
```word
{customer}
    {name}, {email}
    {deliveryAddress.street}, {deliveryAddress.city} {deliveryAddress.postcode} 
{/customer}
```

If there is a Relation with a collection of Entries.

```php
use Santwer\Exporter\Exportable;
...

class User implements HasTemplate {
    use Exportable;
    ...
    protected $exportBlock = 'customer';
    
    public function orders()
    {
        return $this->hasOne(Order::class);
    }
    
    public function deliveryAddress()
    {
        return $this->hasOne(Address::class);
    }

}
```
```word
{customer}
    {name}, {email}
    {orders}
        {orders.product_id} {orders.order_date}
        {deliveryAddress.street}, {deliveryAddress.city} {deliveryAddress.postcode} 
    {/orders}
{/customer}
```

For each Relation it will add up its relation block name.


### Relation Variable with Condition

It is possible to define Variables which are Related to many Entries. Therefore, you can 
reduce it to one relation and get a certain Value in the relation.

It will only select one entry.

```word
{customer}
    {name}, {email}
    Order 15: {orders:15.product_id} {orders:15.order_date}
{/customer}
```

However, you can set up one where condition to get the entry. 
```word
{customer}
    {name}, {email}
    Order 15: {orders:product_id,=,4.product_id} {orders:product_id,=,4.order_date}
{/customer}
```

If the Entry is not found the Values of the Model will be null.
