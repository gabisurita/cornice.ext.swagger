.. _quickstart:

Quickstart
##########

Installing
==========

You may install us with pip::

    $ pip install cornice_swagger


From an existing Cornice application, you may add this extension to your
Pyramid configurator after including cornice:

.. code-block:: python

    from pyramid.config import Configurator

    def setup():
        config = Configurator()
        config.include('cornice')
        config.include('cornice_swagger')


 and create your OpenAPI/Swagger JSON using:

.. code-block:: python

    from cornice_swagger.swagger import CorniceSwagger
    from cornice.service import get_services

    my_generator = CorniceSwagger(get_services())
    my_spec = my_generator('MyAPI', '1.0.0')


Show me a minimalist useful example
===================================

.. literalinclude:: ../../examples/minimalist.py
    :language: python


The resulting `swagger.json` at `http://localhost:8000/__api__` is:

.. code-block:: json

    {
        "swagger": "2.0",
        "info": {
            "version": "1.0.0",
            "title": "MyAPI"
        },
        "basePath": "/",
        "tags": [
            {
                "name": "values"
            },
            {
                "name": "__api__"
            }
        ]
        "paths": {
            "/values/{value}": {
                "parameters": [
                    {
                        "name": "value",
                        "in": "path",
                        "required": true,
                        "type": "string"
                    }
                ],
                "get": {
                    "summary": "Returns the value.",
                    "tags": [
                        "values"
                    ],
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

                    },
                    "produces": [
                        "application/json"
                    ]
                },
                "put": {
                    "summary": "Set the value and returns *True* or *False*.",
                    "tags": [
                        "values"
                    ],
                    "parameters": [
                        {
                            "name": "PutBodySchema",
                            "in": "body",
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
                                "title": "PutBodySchema"
                            },
                            "required": true
                        }
                    ],
                    "produces": [
                        "application/json"
                    ],
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
            },
            "/__api__": {
                "get": {
                    "tags": [
                        "__api__"
                    ],
                    "responses": {
                        "default": {
                            "description": "UNDOCUMENTED RESPONSE"
                        }
                    },
                    "produces": [
                        "application/json"
                    ]
                }
            }
        }
    }

