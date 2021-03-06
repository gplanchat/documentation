Configuration Extensions
========================

Table of Contents
-----------------

-  `Overview <#overview>`__
-  `Creating configuration extension <#creating-configuration-extension>`__
-  `Add options to existing configuration section <#add-options-to-existing-configuration-section>`__
-  `Add new configuration section <#add-new-configuration-section>`__

Overview
--------

Configuration extensions allow to:

-  add new options to existing configuration sections
-  add new configuration sections

Creating configuration extension
--------------------------------

Each configuration extension must implement `ConfigExtensionInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Config/ConfigExtensionInterface.php>`__ (also you can use `AbstractConfigExtension <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Config/AbstractConfigExtension>`__ as a superclass). To register new configuration extension you can add ``Resources/config/oro/app.yml`` in any bundle or use *app/config/config.yml* of your
application. Here is an example:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Oro\Bundle\ApiBundle\Config\AbstractConfigExtension;

    class MyConfigExtension extends AbstractConfigExtension
    {
    }

.. code:: yaml

    # app/config/config.yml
    services:
      acme.api.my_config_extension:
        class: Acme\Bundle\AcmeBundle\Api\MyConfigExtension
        public: false

    oro_api:
        config_extensions:
            - acme.api.my_config_extension

Add options to existing configuration section
---------------------------------------------

To add options to existing configuration section implement the ``getConfigureCallbacks`` method of `ConfigExtensionInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Config/ConfigExtensionInterface.php>`__. Also ``getPreProcessCallbacks`` and ``getPostProcessCallbacks`` methods can be implemented if you need to something before the normalization of during the validation of the configuration.

The following table describes existing sections for which new options can be added.

+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| Section Name                            | When to use                                                                                                                                   |
+=========================================+===============================================================================================================================================+
| entities.entity                         | Add an entity options                                                                                                                         |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| entities.entity.field                   | Add a field options                                                                                                                           |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| relations.entity                        | Add an entity options when the entity is used as a relationship to another entity                                                             |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| relations.entity.field                  | Add a field options when the entity is used as a relationship to another entity                                                               |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| filters                                 | Add options to ``filters`` section                                                                                                            |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| filters.field                           | Add a filter options                                                                                                                          |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| sorters                                 | Add options to ``sorters`` section                                                                                                            |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| sorters.field                           | Add a sorter options                                                                                                                          |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| actions.action                          | Add an action options                                                                                                                         |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| actions.action.status\_code             | Add a response status code options                                                                                                            |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| actions.action.field                    | Add a field options specific for a particular action. These options override options defined in ``entities.entity.field``                     |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| subresources.subresource                | Add a sub-resource options                                                                                                                    |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| subresources.subresource.action         | Add a sub-resource action options                                                                                                             |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| subresources.subresource.action.field   | Add a field options specific for a particular action of a sub-resource. These options override options defined in ``entities.entity.field``   |
+-----------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+

An example:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Symfony\Component\Config\Definition\Builder\NodeBuilder;
    use Oro\Bundle\ApiBundle\Config\AbstractConfigExtension;

    class MyConfigExtension extends AbstractConfigExtension
    {
        /**
         * {@inheritdoc}
         */
        public function getConfigureCallbacks()
        {
            return [
                'entities.entity' => function (NodeBuilder $node) {
                    $node->scalarNode('some_option');
                }
            ];
        }

        /**
         * {@inheritdoc}
         */
        public function getPreProcessCallbacks()
        {
            return [
                'entities.entity' => function (array $config) {
                    // do something
                    return $config;
                }
            ];
        }

        /**
         * {@inheritdoc}
         */
        public function getPostProcessCallbacks()
        {
            return [
                'entities.entity' => function (array $config) {
                    // do something
                    return $config;
                }
            ];
        }
    }

Add new configuration section
-----------------------------

To add new configuration section you need to create a class implements `ConfigurationSectionInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Config/Definition/ConfigurationSectionInterface.php>`__ and return instance of it in the ``getEntityConfigurationSections`` method of your configuration extension. By default the configuration will be returned as an array, but if you want to provide a class represents configuration of your section you can implement a
configuration loader. The loader is a class implements `ConfigLoaderInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Config/ConfigLoaderInterface.php>`__. An instance of the loader should be returned by the ``getEntityConfigurationLoaders`` method of your configuration extension.

An example of simple configuration section:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Symfony\Component\Config\Definition\Builder\NodeBuilder;
    use Oro\Bundle\ApiBundle\Config\Definition\AbstractConfigurationSection;

    class MyConfiguration extends AbstractConfigurationSection
    {
        public function configure(NodeBuilder $node)
        {
            $node->scalarNode('some_option');
        }
    }

An example of a configuration section that can be extended by other bundles:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Symfony\Component\Config\Definition\Builder\ArrayNodeDefinition;
    use Symfony\Component\Config\Definition\Builder\NodeBuilder;
    use Oro\Bundle\ApiBundle\Config\Definition\AbstractConfigurationSection;

    class MyConfiguration extends AbstractConfigurationSection
    {
        public function configure(NodeBuilder $node)
        {
            $sectionName = 'my_section';

            /** @var ArrayNodeDefinition $parentNode */
            $parentNode = $node->end();
            $this->callConfigureCallbacks($node, $sectionName);
            $this->addPreProcessCallbacks($parentNode, $sectionName);
            $this->addPostProcessCallbacks($parentNode, $sectionName);

            $node->scalarNode('some_option');
        }
    }

An example of configuration section loader:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Oro\Bundle\ApiBundle\Config\AbstractConfigLoader;

    class MyConfigLoader extends AbstractConfigLoader
    {
        /**
         * {@inheritdoc}
         */
        public function load(array $config)
        {
            $result = new MyConfigSection();
            foreach ($config as $key => $value) {
                $this->loadConfigValue($result, $key, $value);
            }

            return $result;
        }
    }

The configuration extension:

.. code:: php

    <?php
    namespace Acme\Bundle\AcmeBundle\Api;

    use Oro\Bundle\ApiBundle\Config\AbstractConfigExtension;

    class MyConfigExtension extends AbstractConfigExtension
    {
        /**
         * {@inheritdoc}
         */
        public function getEntityConfigurationSections()
        {
            return ['my_section' => new MyConfiguration()];
        }

        /**
         * {@inheritdoc}
         */
        public function getEntityConfigurationLoaders()
        {
            return ['my_section' => new MyConfigLoader()];
        }
    }

An example of usage created configuration section:

.. code:: yaml

    api:
        ...
        entities:
            Acme\Bundle\AcmeBundle\Entity\AcmeEntity:
                my_section:
                    my_option: value

To check that you configuration section was added correctly run ``php bin/console oro:api:config:dump-reference``. The output will be something like this:

.. code:: yaml

    # The structure of "Resources/config/oro/api.yml"
    api:
        ...
        entities:
            name:
                ...
                my_section:
                    my_option: ~
                ...
