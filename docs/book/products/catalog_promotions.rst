.. index::
   single: Catalog Promotions

Catalog Promotions
==================

The **Catalog Promotions** system in **Sylius** is a new way of dealing with promotions on multiple products.
If you get used to `Cart Promotions </book/orders/cart-promotions>` this will be something familiar to you.

It is managed by combination of promotion rules and actions, where you can specify on which e.g. products or taxons
you can specify the Catalog Promotion with your custom actions as well as actions like percentage discount.

You can assign the needed channels too.

Catalog Promotion Parameters
----------------------------

Catalog Promotion has a few basic parameters that represent it - a unique ``code`` and ``name``:

.. note::

    The parameter ``code`` should contain only letters, numbers, dashes and underscores (like all codes in Sylius).
    We encourage you to use ``snake_case`` codes.

.. code-block:: bash

    {
        "code": "t_shirt_promotion" # unique
        "name": "T-shirt Promotion"
        # ...
    }

Rest of the fields are used for configuration:

* **Channels** are used to define channels on which given promotion is applied:

.. code-block:: bash

    {
        #...
        "channels": [
            "/api/v2/admin/channels/FASHION_WEB", #IRI
            "/api/v2/admin/channels/HOME_WEB"
        ]
        # ...
    }

* **Rules** are used to define rules on which the catalog promotion will work:

.. code-block:: bash

    {
        #...
        "rules": [
            {
                "type": "for_variants",
                "configuration": {
                    "variants": [
                        "Everyday_white_basic_T_Shirt-variant-1", #Variant Code
                        "Everyday_white_basic_T_Shirt-variant-4"
                    ]
                }
            }
        ]
        # ...
    }

.. note::

    The usage of Variant Code over IRI is in this case dictated by the kind of relationship.
    Here it is a part of configuration, where e.g. channel is a relation to the resource.

For possible rules see `Catalog Promotion Rules configuration reference`_

* **Actions** are used to defined what happens when the promotion is applied:

.. code-block:: bash

    {
        #...
        "actions": [
            {
                "type": "percentage_discount",
                "configuration": {
                    "amount": 0.5 #float
                }
            }
        ]
        # ...
    }

* **Translations** are used to define labels and descriptions for languages you are configuring:

.. code-block:: bash

    {
        #...
        "translations": {
            "en_US": {
                "label": "Summer discount",
                "description": "The grass so green, the sun so bright. Life seems a dream, no worries in sight.",
                "locale": "en_US" #Locale Code
                }
            }
        }
        # ...
    }

How to create a Catalog Promotion?
----------------------------------

After we get to know with some basics of Catalog Promotion let's see how we can create one:

* **API** The common use case is to make it through API, first you need to authorize yourself as an admin (you don't want to let a guest create it - don't you?).

.. tip::

    Check this doc `Authorization </book/api/authorization>` if you are having trouble with login in.

And let's call the POST endpoint to create very basic catalog promotion:

.. code-block:: bash

    curl -X 'POST' \
      'https://hostname/api/v2/admin/catalog-promotions' \
      -H 'accept: application/ld+json' \
      -H 'Authorization: Bearer authorizationToken' \
      -H 'Content-Type: application/ld+json' \
      -d '{
        "code": "t_shirt_promotion",
        "name": "T-shirt Promotion"
        }'

If everything was fine, the server will respond with 201 status code.
This means you have created a simple catalog promotion with ``name`` and ``code`` only.

You can check if the catalog promotion exists by using GET endpoint

.. code-block:: bash

    curl -X 'GET' \
    'https://hostname/api/v2/admin/catalog-promotions'

* **Programmatically** Similar to cart promotions you can use factory to create a new catalog promotion:

.. code-block:: php

   /** @var CatalogPromotionInterface $promotion */
   $promotion = $this->container->get('sylius.factory.t_shirt_promotion')->createNew();

   $promotion->setCode('t_shirt_promotion');
   $promotion->setName('T-shirt Promotion');

.. note::

    Take into account that both the API and Programmatically added catalog promotions in this shape are not really useful.
    You need to add configurations to them so they make any business valued changes.

How to create a Catalog Promotion Rule and Action?
--------------------------------------------------

The creation of Catalog Promotion was quite simple but at this shape it has no real functionality. Let's add rule and action:

In API we will extend last command:

.. code-block:: bash

    curl -X 'POST' \
      'https://hostname/api/v2/admin/catalog-promotions' \
      -H 'accept: application/ld+json' \
      -H 'Authorization: Bearer authorizationToken' \
      -H 'Content-Type: application/ld+json' \
      -d '{
        "code": "t_shirt_promotion",
        "name": "T-shirt Promotion",
        "channels": [
            "/api/v2/admin/channels/FASHION_WEB"
        ],
        "rules": [
            {
              "type": "for_variants",
              "configuration": {
                "variants": ["Everyday_white_basic_T_Shirt-variant-1", "Everyday_white_basic_T_Shirt-variant-4"]
              }
            }
          ],
          "actions": [
            {
              "type": "percentage_discount",
              "configuration": {
                "amount": 0.5
              }
            }
          ],
          "translations": {
            "en_US": {
              "label": "T-shirt Promotion",
              "description": "T-shirt Promotion description",
              "locale": "en_US"
            }
        }'

This will create a catalog promotions with relations to Rule ``for_variants``, Action ``percentage_discount`` and also
translation for ``en_US`` locale.

We can also make it programmatically:

.. code-block:: php

    /** @var CatalogPromotionInterface $catalogPromotion */
    $catalogPromotion = $this->container->get('sylius.factory.catalog_promotion')->createNew();
    $catalogPromotion->setCode('t_shirt_promotion');
    $catalogPromotion->setName('T-shirt Promotion');

    $catalogPromotion->setCurrentLocale('en_US');
    $catalogPromotion->setFallbackLocale('en_US');
    $catalogPromotion->setLabel('T-shirt Promotion');
    $catalogPromotion->setDescription('T-shirt Promotion description');

    $catalogPromotion->addChannel('FASHION_WEB');

    /** @var CatalogPromotionRuleInterface $catalogPromotionRule */
    $catalogPromotionRule = $this->catalogPromotionRuleExampleFactory->create($rule);
    $catalogPromotionRule->setCatalogPromotion($catalogPromotion);
    $catalogPromotion->addRule($catalogPromotionRule);

    /** @var CatalogPromotionActionInterface $catalogPromotionAction */
    $catalogPromotionAction = $this->catalogPromotionActionExampleFactory->create($action);
    $catalogPromotionAction->setCatalogPromotion($catalogPromotion);
    $catalogPromotion->addAction($catalogPromotionAction);

And now you should be able to see created Catalog Promotion. You can check if it exists like in the last example (with GET endpoint).
If you look into ``product-variant`` endpoint in shop you should see now that chosen variants have lowered price and added field ``appliedPromotions``:

.. code-block:: bash

    curl -X 'GET' \
    'https://hostname/api/v2/shop/product-variant/Everyday_white_basic_T_Shirt-variant-1'

.. code-block:: bash

    # response content
    {
        "@context": "/api/v2/contexts/ProductVariant",
        "@id": "/api/v2/shop/product-variants/Everyday_white_basic_T_Shirt-variant-1",
        # ...
        "price": 2000,
        "originalPrice": 4000,
        "appliedPromotions": {
            "T-shirt Promotion": {
                "name": "T-shirt Promotion",
                "description": "T-shirt Promotion description"
            }
        },
        "inStock": true
    }

Catalog Promotion Rules configuration reference
'''''''''''''''''''''''''''''''''''''''''''''''

+-------------------------------+--------------------------------------------------------------------+
| Rule type                     | Rule Configuration Array                                           |
+===============================+====================================================================+
| ``for_variants``              | ``['variants' => [$variantCodes]]``                                |
+-------------------------------+--------------------------------------------------------------------+

Catalog Promotion Actions configuration reference
'''''''''''''''''''''''''''''''''''''''''''''''''

+-------------------------------+--------------------------------------------------------------------+
| Action type                   | Action Configuration Array                                         |
+===============================+====================================================================+
| ``percentage_discount``       | ``['amount' => $amountFloat]``                                     |
+-------------------------------+--------------------------------------------------------------------+

Learn more
----------

* :doc:`Cart Promotions </book/orders/cart-promotions>`