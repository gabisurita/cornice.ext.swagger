.. _tutorial:

Full Tutorial
#############

Here you may find the general aspects used by Colander Swagger to generate
the swagger documentation. Most examples presented on this section refer
to the example on `quickstart`.

Using the Pyramid Hook
======================

In order to enable response documentation, you must add this extension to
your Pyramid config. For that you may use:


.. code-block:: python

    from pyramid.config import Configurator

    def setup():
        config = Configurator()
        config.include('cornice')
        config.include('cornice_swagger')


If you don't know what this is about or need more information, please check the
`Pyramid documentation <http://docs.pylonsproject.org/projects/pyramid>`_

Generating summaries with view docstrings
=========================================

You may use view docstrings to create operation summaries. For example,
the following view definition docstring will correspond to the following
swagger summary:

.. code-block:: python

    values = Service(name='foo',
                     path='/values')

    @values.get()
    def get_value(request):
        """Returns the value."""


.. code-block:: json

    {
        "paths": {
            "/values": {
                "get": {
                    "summary": "Returns the value."
                }
            }
        }
    }


Extracting path parameters
==========================

Path parameters may be automatically extracted from the service definition,
you may overwrite then with schemas on the view if you wish to add more
view-specific information.

Another example:

.. code-block:: python

    values = Service(name='foo',
                     path='/values/{value}')


.. code-block:: json

    {
        "paths": {
            "/values/{value}": {
                "parameters": [
                    {
                        "name": "value",
                        "in": "path",
                        "required": true,
                        "type": "string"
                    }
                ]
            }
        }
    }


Extracting parameters from cornice schemas
==========================================

When using colander validators such as ``colader_validator`` or
``colander_body_validator``, we can extract the operation parameters
from the request schema. The schemas should comply with
`Cornice 2.0 colander schemas <https://cornice.readthedocs.io/en/latest/schema.html#multiple-request-attributes>`_.


.. code-block:: python

    from cornice.validators import colander_body_validator

    values = Service(name='foo',
                     path='/values/{value}')

    class PutBodySchema(colander.MappingSchema):
        value = colander.SchemaNode(colander.String(),
                                    description='My precious value')


    @values.put(validators=(colander_body_validator, ),
                schema=PutBodySchema())
    def set_value(request):
        """Set the value and returns *True* or *False*."""


.. code-block:: json

    {
        "paths": {
            "/values/{value}": {
                "put": {
                    "parameters": [
                        {
                            "name": "PutBodySchema",
                            "in": "body",
                            "required": true,
                            "schema": {
                                "title": "PutBodySchema",
                                "type": "object",
                                "properties": {
                                    "value": {
                                        "type": "string",
                                        "description": "My precious value",
                                        "title": "Value"
                                    }
                                },
                                "required": [
                                    "value"
                                ]
                            }
                        }
                    ]
                }
            }
        }
    }


When using `colander_validator`, the request should have fields corresponding
the parameters locations as follows:


.. code-block:: python

    from cornice.validators import colander_validator

    class BodySchema(colander.MappingSchema):
        value = colander.SchemaNode(colander.String(),
                                    description='My precious value')


    class QuerySchema(colander.MappingSchema):
        foo = colander.SchemaNode(colander.String(), missing=colander.drop)


    class HeaderSchema(colander.MappingSchema):
        bar = colander.SchemaNode(colander.String(), default='blah')


    class PutRequestSchema(colander.MappingSchema):
        body = BodySchema()
        querystring = QuerySchema()
        header = HeaderSchema()


    @values.put(validators=(colander_validator, ),
                schema=PutRequestSchema())
    def set_value(request):
        """Set the value and returns *True* or *False*."""


Documenting responses
=====================

Unfortunately, on Cornice we don't have a way to provide response schemas, so
this part must be provided separately and handled by Cornice Swagger.

For that you must provide a Response Colander Schema that follows the pattern:

.. code-block:: python

    class ResponseSchema(colander.MappingSchema):
        body = BodySchema()
        headers = HeaderSchema()


    get_response_schemas = {
        '200': ResponseSchema(description='Return my OK response'),
        '404': ResponseSchema(description='Return my not found response')
    }


Notice that the ``ResponseSchema`` class follows the same pattern as the
Cornice requests using ``cornice.validators.colander_validator``
(except for querystrings, since obviously we don't have querystrings on responses).

A response schema mapping, as the ``get_response_schemas`` dict should aggregate
response schemas as the one defined as ``ResponseSchema`` with keys matching the
response status code of for each entry. All schema entries should contain descriptions.
You may also provide a ``default`` response schema to be used if the response doesn't
match any of the status codes provided.

From our minimalist example:


.. code-block:: python

    values = Service(name='foo',
                     path='/values/{value}')

    # Create a body schema for our requests
    class BodySchema(colander.MappingSchema):
        value = colander.SchemaNode(colander.String(),
                                    description='My precious value')


    # Create a response schema for our 200 responses
    class OkResponseSchema(colander.MappingSchema):
        body = BodySchema()


    # Aggregate the response schemas for get requests
    response_schemas = {
        '200': OkResponseSchema(description='Return value')
    }


    @values.put(response_schemas=response_schemas)
    def set_value(request):
        """Set the value and returns *True* or *False*."""


.. code-block:: json

    {
        "paths": {
            "/values/{value}": {
                "put": {
                    "responses": {
                        "200": {
                            "description": "Return value",
                            "schema": {
                                "required": [
                                    "value"
                                ],
                                "type": "object",
                                "properties": {
                                    "value": {
                                        "type": "string",
                                        "description": "My precious value",
                                        "title": "Value"
                                    }
                                },
                                "title": "BodySchema"
                            }
                        }
                    }
                }
            }
        }
    }
