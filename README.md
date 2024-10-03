To add another parameter to additional attributes in Magento, you'll need to create a new module or modify an existing one. I'll guide you through creating a new module to add a custom attribute to product options. Let's call this module "Acme_CustomOptionAttribute".
Here's a step-by-step tutorial:

Create the module structure:

```bash
Acme_CustomOptionAttribute/
│
├── registration.php
├── etc/
│   ├── module.xml
│   ├── db_schema.xml
│   └── di.xml
│
├── Model/
│   └── Product/
│       └── Option/
│           └── Value/
│               └── CustomAttribute.php
│
├── Setup/
│   └── Patch/
│       └── Data/
│           └── AddCustomAttributeToOptionValue.php
│
└── Ui/
    └── DataProvider/
        └── Product/
            └── Form/
                └── Modifier/
                    └── CustomOptionAttribute.php
```

Create the `registration.php` file:

```php
<?php

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    'Acme_CustomOptionAttribute',
    __DIR__
);
```

Create the `etc/module.xml` file:

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Acme_CustomOptionAttribute" setup_version="1.0.0">
        <sequence>
            <module name="Magento_Catalog"/>
        </sequence>
    </module>
</config>
```


4. Create the `etc/db_schema.xml` file to add the new attribute:

```xml
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="catalog_product_option_type_value" resource="default" engine="innodb">
        <column xsi:type="varchar" name="acme_custom_attribute" nullable="true" length="255" comment="Acme Custom Attribute"/>
    </table>
</schema>
```

5. Create the `Model/Product/Option/Value/CustomAttribute.php` file:

```php
<?php
namespace Acme\CustomOptionAttribute\Model\Product\Option\Value;

use Magento\Framework\Model\AbstractModel;

class CustomAttribute extends AbstractModel
{
    protected function _construct()
    {
        $this->_init(\Acme\CustomOptionAttribute\Model\ResourceModel\Product\Option\Value\CustomAttribute::class);
    }
}
```

6. Create the `Setup/Patch/Data/AddCustomAttributeToOptionValue.php` file:

```php
<?php
namespace Acme\CustomOptionAttribute\Setup\Patch\Data;

use Magento\Framework\Setup\Patch\DataPatchInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class AddCustomAttributeToOptionValue implements DataPatchInterface
{
    private $moduleDataSetup;

    public function __construct(ModuleDataSetupInterface $moduleDataSetup)
    {
        $this->moduleDataSetup = $moduleDataSetup;
    }

    public function apply()
    {
        $this->moduleDataSetup->startSetup();

        $this->moduleDataSetup->getConnection()->addColumn(
            $this->moduleDataSetup->getTable('catalog_product_option_type_value'),
            'acme_custom_attribute',
            [
                'type' => \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
                'nullable' => true,
                'comment' => 'Acme Custom Attribute'
            ]
        );

        $this->moduleDataSetup->endSetup();
    }

    public static function getDependencies()
    {
        return [];
    }

    public function getAliases()
    {
        return [];
    }
}
```

7. Create the `Ui/DataProvider/Product/Form/Modifier/CustomOptionAttribute.php` file:

```php
<?php
namespace Acme\CustomOptionAttribute\Ui\DataProvider\Product\Form\Modifier;

use Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\AbstractModifier;
use Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\CustomOptions;
use Magento\Ui\Component\Form\Element\Input;
use Magento\Ui\Component\Form\Field;

class CustomOptionAttribute extends AbstractModifier
{
    public function modifyMeta(array $meta)
    {
        $groupCustomOptionsName = CustomOptions::GROUP_CUSTOM_OPTIONS_NAME;
        $optionContainerName = CustomOptions::CONTAINER_OPTION;

        if (!empty($meta[$groupCustomOptionsName])) {
            $meta[$groupCustomOptionsName]['children']['options']['children']['record']['children']
            [$optionContainerName]['children']['values']['children']['record']['children']['acme_custom_attribute'] = $this->getCustomAttributeConfig();
        }

        return $meta;
    }

    protected function getCustomAttributeConfig()
    {
        return [
            'arguments' => [
                'data' => [
                    'config' => [
                        'label' => __('Acme Custom Attribute'),
                        'componentType' => Field::NAME,
                        'formElement' => Input::NAME,
                        'dataScope' => 'acme_custom_attribute',
                        'dataType' => 'text',
                        'sortOrder' => 250,
                    ],
                ],
            ],
        ];
    }

    public function modifyData(array $data)
    {
        return $data;
    }
}
```


8. `Create the etc/di.xml file to add the UI component:`

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <virtualType name="Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\Pool">
        <arguments>
            <argument name="modifiers" xsi:type="array">
                <item name="acme_custom_attribute" xsi:type="array">
                    <item name="class" xsi:type="string">Acme\CustomOptionAttribute\Ui\DataProvider\Product\Form\Modifier\CustomOptionAttribute</item>
                    <item name="sortOrder" xsi:type="number">70</item>
                </item>
            </argument>
        </arguments>
    </virtualType>
</config>
```













