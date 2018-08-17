# Collections

This section was created to show how collections can be fetched in a faster way.

## Instantiating Collection Objects

Rather than use a model to instantiate its collection object use a method to instantiate a collection object directly.

### Wrong Way
```php
/** Aways retrieve a NEW model instance. */
$collection = Mage::getModel('catalog/product')->getCollection();
```

### Right Way
```php
/** Aways retrieve a NEW model instance. */
$collection = Mage::getResourceModel('catalog/product_collection');

/** Aways retrieve a SINGLETON model instance. */
$collection = Mage::getResourceSingleton('catalog/product_collection');
```

## Fetching Collections Faster

By default, when you iterate a collection its method `load()` is called and, internally, for each row from the result set a new data model is instantiated. See below:

```php
/** @var Mage_Catalog_Model_Resource_Product_Collection $collection */
$collection = Mage::getResourceModel('catalog/product_collection');

/** @var Mage_Catalog_Model_Product $product */
foreach ($collection as $product) {
  /** The $product variable is an instance of Mage_Catalog_Model_Catalog */
  ...
}
```

Inside the foreach above, for each time the collection is iterated a product's model is passed to the context. Where this `Mage_Catalog_Model_Product` class comes from?

Take a look at the method right below:

```php
class Varien_Data_Collection_Db extends Varien_Data_Collection
{

    ...

    public function load($printQuery = false, $logQuery = false)
    {
        if ($this->isLoaded()) {
            return $this;
        }

        $this->_beforeLoad();

        $this->_renderFilters()
             ->_renderOrders()
             ->_renderLimit();

        $this->printLogQuery($printQuery, $logQuery);
        $data = $this->getData();
        $this->resetData();

        if (is_array($data)) {
            foreach ($data as $row) {
                $item = $this->getNewEmptyItem();
                if ($this->getIdFieldName()) {
                    $item->setIdFieldName($this->getIdFieldName());
                }
                $item->addData($row);
                $this->addItem($item);
            }
        }

        $this->_setIsLoaded();
        $this->_afterLoad();
        return $this;
    }
    
    ...
    
}
```

The data models come from this exactly part:

```php
    ...
    
    if (is_array($data)) {
        foreach ($data as $row) {
            $item = $this->getNewEmptyItem();
            if ($this->getIdFieldName()) {
                $item->setIdFieldName($this->getIdFieldName());
            }
            $item->addData($row);
            $this->addItem($item);
        }
    }
    
    ...
```

Basically, when you call the `load()` method from a Collection object, which is an instance from `Varien_Data_Collection_Db`.


There's a way for fetching a collection much faster than simply running `load()` method for the collection classes. This way is a good aproach when you don't need the instances of the data objects as a result when iterating the collection.
By default, when you iterate a collection its method `load()` is called and, internally, for each row from the result set a new data model is instantiated
