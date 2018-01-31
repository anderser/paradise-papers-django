==========================
How to serialize the data.
==========================

Why we need to serialize the data
=================================

The nodes that we are fetching so far are just instances of Python objects, we want to return JSON files to the fetch calls that are going to be made in the frontend, Python objects structure is not JSON serializable and the nodes don't support the already build in serializers on the library, so we have to create our own serialize methods and manage the logic of those since sometimes we would need to serialize only the directly related nodes to a specific node, to serialize the dataa is nothing more than getting it from the object fromat to a parseable format.

Create serialize methods
========================

The way we are going to serialize the data is using property methods on each model, to then call them on the helper functions. We are going to be using the ``Entity`` model for these examples since this one presents all the cases contemplated in this tutorial.

In your ``fetch_api/models/entity.py`` add the following as a method of the ``Entity`` class::

    @property
    def serialize(self):
        return {
            'node_properties': {
                'sourceID': self.sourceID,
                'address': self.address,
                'jurisdiction': self.jurisdiction,
                'service_provider': self.service_provider,
                'countries': self.countries,
                'jurisdiction_description': self.jurisdiction_description,
                'valid_until': self.valid_until,
                'ibcRUC': self.ibcRUC,
                'name': self.name,
                'country_codes': self.country_codes,
                'incorporation_date': self.incorporation_date,
                'node_id': self.node_id,
                'status': self.status,
            },
        }

If you look at this closely you'll notice this is just all the properties on the ``Entity`` class mapped to a dictionary but the relationships, we'll explain later why we don't serialize the relationships here and how to do it, you will need to do the same for the other model classes:

Address::

    @property
    def serialize(self):
        return {
            'node_properties': {
                'sourceID': self.sourceID,
                'country_codes': self.country_codes,
                'valid_until': self.valid_until,
                'address': self.address,
                'countries': self.countries,
                'node_id': self.node_id,
            },
        }

Intermediary::

    @property
    def serialize(self):
        return {
            'node_properties': {
                'sourceID': self.sourceID,
                'valid_until': self.valid_until,
                'name': self.name,
                'country_codes': self.country_codes,
                'countries': self.countries,
                'node_id': self.node_id,
                'status': self.status,
            },
        }

Officer::

    @property
    def serialize(self):
        return {
            'node_properties': {
                'sourceID': self.sourceID,
                'name': self.name,
                'country_codes': self.country_codes,
                'valid_until': self.valid_until,
                'countries': self.countries,
                'node_id': self.node_id,
            },
        }

Other::

    @property
    def serialize(self):
        return{
            'node_properties': {
                'sourceID': self.sourceID,
                'name': self.name,
                'countries': self.countries,
                'valid_until': self.valid_until,
                'node_id': self.node_id,
            },
        }

Call the serialize methods on the helpers
=========================================

Now instead of your ``fetch_nodes`` function on your ``fetch_api/utils.py`` looking like this::

    def fetch_nodes(fetch_info):
        node_type       = fetch_info['node_type']
        search_word     = fetch_info['name']
        country         = fetch_info['country']
        limit           = fetch_info['limit']
        skip            = ((fetch_info['skip'] - 1) * limit)
        jurisdiction    = fetch_info['jurisdiction']
        node_set        = filter_nodes(MODEL_ENTITIES[node_type], search_word, country, jurisdiction)
        node_set.limit  = limit
        node_set.skip   = skip
        fetched_nodes   = node_set.all()

        return fetched_nodes

Change the return statement to::

        return [node.serialize for node in fetched_nodes]

That's just going to create a new array of dictionaries with the values of the serialized nodes.

And your ``fetch_node_details`` function looking like this::

    def fetch_node_details(node_info):
        node_type       = node_info['node_type']
        node_id         = node_info['node_id']
        node            = MODEL_ENTITIES[node_type].nodes.get(node_id=node_id)

        return node

Should look like this::

    def fetch_node_details(node_info):
        node_type       = node_info['node_type']
        node_id         = node_info['node_id']
        node            = MODEL_ENTITIES[node_type].nodes.get(node_id=node_id)
        node_details    = node.serialize

        return node_details

Basically doing the same as before but for a single node instead of a set.

Create the serialize relationships methods
==========================================

We are not serializing the relationships along with the properties because:

* That would create a loophole between nodes
* You don't always need the relationships

So, with this said, this is how we are gonna serialize the relationships, add the following to the file ``fetch_api/models/entity.py``::

    from .nodeutils import NodeUtils

And to your ``Entity`` class::

    class Entity(StructuredNode, NodeUtils):

    ...

    @property
    def serialize_connections(self):
        return [
            {
                'nodes_type': 'Officer',
                'nodes_related': self.serialize_relationships(self.officers.all(), 'OFFICER_OF'),
            },
            {
                'nodes_type': 'Intermediary',
                'nodes_related': self.serialize_relationships(self.intermediaries.all(), 'INTERMEDIARY_OF'),
            },
            {
                'nodes_type': 'Address',
                'nodes_related': self.serialize_relationships(self.addresses.all(), 'REGISTERED_ADDRESS'),
            },
            {
                'nodes_type': 'Other',
                'nodes_related': self.serialize_relationships(self.others.all(), 'CONNECTED_TO'),
            },
            {
                'nodes_type': 'Entity',
                'nodes_related': self.serialized_realtionships_of_type('Entity'),
            },
        ]

You will need to do the same for the other model classes:

Address::

    @property
    def serialize_connections(self):
        return [
            {
                'nodes_type': 'Officer',
                'nodes_related': self.serialize_relationships(self.officers.all(), 'REGISTERED_ADDRESS'),
            },
            {
                'nodes_type': 'Intermediary',
                'nodes_related': self.serialize_relationships(self.intermediaries.all(), 'REGISTERED_ADDRESS'),
            },
    ]

Intermediary::

    @property
    def serialize_connections(self):
        return [
            {
                'nodes_type': 'Entity',
                'nodes_related': self.serialize_relationships(self.entities.all(), 'INTERMEDIARY_OF'),
            },
            {
                'nodes_type': 'Address',
                'nodes_related': self.serialize_relationships(self.addresses.all(), 'REGISTERED_ADDRESS'),
            },
            {
                'nodes_type': 'Officer',
                'nodes_related': self.serialized_realtionships_of_type('Officer'),
            },
        ]

Officer::

    @property
    def serialize_connections(self):
        return [
            {
                'nodes_type': 'Address',
                'nodes_related': self.serialize_relationships(self.addresses.all(), 'REGISTERED_ADDRESS'),
            },
            {
                'nodes_type': 'Entity',
                'nodes_related': self.serialize_relationships(self.entities.all(), 'OFFICER_OF'),
            },
            {
                'nodes_type': 'Officer',
                'nodes_related': self.serialized_realtionships_of_type('Officer'),
            },
        ]

Other::

    @property
    def serialize_connections(self):
        return [
            {
                'nodes_type': 'Officer',
                'nodes_related': self.serialized_realtionships_of_type('Officer'),
            },
            {
                'nodes_type': 'Entity',
                'nodes_related': self.serialized_realtionships_of_type('Entity'),
            },
            {
                'nodes_type': 'Address',
                'nodes_related': self.serialize_relationships(self.addresses.all(), 'REGISTERED_ADDRESS'),
            },
        ]


Call the serialize relationships methods on the helpers
=======================================================

On your ``fetch_api/utils.py`` the ``fetch_node_details`` function change put this above the return statement::

        # Make sure to return an empty array if not connections
        node_details['node_connections'] = []
        if (hasattr(node, 'serialize_connections')):
            node_details['node_connections'] = node.serialize_connections

So the function should be looking something like this::

    def fetch_node_details(node_info):
        node_type       = node_info['node_type']
        node_id         = node_info['node_id']
        node            = MODEL_ENTITIES[node_type].nodes.get(node_id=node_id)
        node_details    = node.serialize

        # Make sure to return an empty array if not connections
        node_details['node_connections'] = []
        if (hasattr(node, 'serialize_connections')):
            node_details['node_connections'] = node.serialize_connections

        return node_details

Return the json to the frontend
===============================

Now if you call your functions ``fetch_node_details`` and ``fetch_nodes`` should be returning the same data but in a way that is JSON parsable, so let's change a couple things in order of returning this data that we need.

In your project settings file for us it is ``paradise_papers_search/settings/base.py`` but if you are using a costume project it may be different for you, add the following:

    # Rest-Framework settings
    REST_FRAMEWORK = {
        'DEFAULT_RENDERER_CLASSES': (
            'rest_framework.renderers.JSONRenderer',
        )
    }

This is going to cause that when we invoke the render method on our APIs views the method will call the JSON renderer rather than just the HTTP one.

So once again we will clean our ``fetch_api/views.py`` file and leave it like this::

    from rest_framework.views import APIView
    from rest_framework.response import Response
    from .models.helpers import fetch_nodes

    class GetNodesData(APIView):
        def get(self, request):
            fetch_info = {
                'node_type': request.GET.get('t', 'Entity'),
                'name': request.GET.get('q', ''),
                'country': request.GET.get('c', ''),
                'jurisdiction': request.GET.get('j', ''),
                'sourceID': request.GET.get('s', ''),
                'limit': 10,
                'skip': int(request.GET.get('p', 1)),
            }
            nodes = fetch_nodes(fetch_info)
            data = {
                'response': {
                    'status': '200',
                    'rows': len(nodes),
                    'data': nodes,
                },
            }
            return Response(data)

Here we are just taking the query parameters off the request and parsing them to pass them to the respective fetch function.

Now if you try this:

``curl http://127.0.0.1:8000/fetch/nodes?q=maria&t=Entity``

You should see something like this::

    {"response":{"data":[{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"MARIANTHI LIMITED","jurisdiction_description":"Seychelles","service_provider":"Mossack Fonseca","incorporation_date":"14-JUL-2009","countries":"United Arab Emirates","country_codes":"ARE","ibcRUC":"063736","address":"OMNI MANAGEMENT CONSULTANCY FZE OFFICE NUMBER 425; RAS AL KHAIMAH FREE TRADE ZONE AUTHORITY  GOVERNMENT OF RAS AL KHAIMAH; P.O. BOX 10055 RAS AL KHAIMAH; UNITED ARAB EMIRATES","status":"Defaulted","node_id":"10026610","jurisdiction":"SEY","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"LUZMARIA S.A.","jurisdiction_description":"Seychelles","service_provider":"Mossack Fonseca","incorporation_date":"24-JAN-2013","countries":"Luxembourg","country_codes":"LUX","ibcRUC":"118634","address":"EFG BANK (LUXEMBOURG9 S.A. ATTN: PIERRE AVIRON-VIOLET; 14, ALLÉE MARCONI; L - 2013 LUXEMBOURG  LUXEMBOURG","status":"Active","node_id":"10027827","jurisdiction":"SEY","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"NUMMARIA LIMITED","jurisdiction_description":"Niue","service_provider":"Mossack Fonseca","incorporation_date":"23-OCT-1997","countries":"United Kingdom","country_codes":"GBR","ibcRUC":"002351","address":"HOLLINGWORTH CONSULTANTS LTD. PARKVIEW HOUSE BUCCLEUCH ROAD HAWICK; ROXBURGHSHIRE SCOTLAND; TD9 0EL","status":"Defaulted","node_id":"10036241","jurisdiction":"NIUE","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"GRUPO NUMMARIA S.L.","jurisdiction_description":"Niue","service_provider":"Mossack Fonseca","incorporation_date":"12-FEB-1996","countries":"United Kingdom","country_codes":"GBR","ibcRUC":"000737","address":"HOLLINGWORTH CONSULTANTS LTD. PARKVIEW HOUSE BUCCLEUCH ROAD HAWICK; ROXBURGHSHIRE SCOTLAND; TD9 0EL","status":"Defaulted","node_id":"10036779","jurisdiction":"NIUE","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"MARIACHI CORP.","jurisdiction_description":"Niue","service_provider":"Mossack Fonseca","incorporation_date":"09-AUG-1999","countries":"Belize","country_codes":"BLZ","ibcRUC":"004700","address":"BOND & COMPANY 35 BARRACK ROAD BELIZE CITY BELIZE*S.I.*","status":"Defaulted","node_id":"10040810","jurisdiction":"NIUE","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"M.P. MARIANNE S.A.","jurisdiction_description":"Panama","service_provider":"Mossack Fonseca","incorporation_date":"02-AUG-2007","countries":"Switzerland","country_codes":"CHE","ibcRUC":"51","address":"UNION BANCAIRE PRIVÉE UBP (SWITZERLAND) ATTN: MR. FABIEN DE FRAIPONT RUE DU RHÔNE 96-98  CP 1320 CH-1211 GENEVA 1 SWITZERLAND GENEVE SWITZERLAND","status":"Changed agent","node_id":"10053581","jurisdiction":"PMA","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"MARIADA HOLDINGS LIMITED","jurisdiction_description":"British Virgin Islands","service_provider":"Mossack Fonseca","incorporation_date":"21-JUL-1995","countries":"Switzerland","country_codes":"CHE","ibcRUC":"156189","address":"PRIMEWAY S.A. 7, RUE DU RHÔNE 1204 GENEVE SWITZERLAND","status":"Changed agent","node_id":"10064371","jurisdiction":"BVI","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"MARIANNE PROPERTIES LIMITED","jurisdiction_description":"British Virgin Islands","service_provider":"Mossack Fonseca","incorporation_date":"02-JUL-1992","countries":"Guernsey","country_codes":"GGY","ibcRUC":"65048","address":"KLEINWORT BENSON (GUERNSEY) TRUSTEES LIMITED P.O. BOX 44   WESTBOURNE; THE GRANGE ST. PETER PORT; GUERNSEY GY1 3BG CHANNEL ISLANDS ATTN: MS. TINA BROWNING","status":"Defaulted","node_id":"10057577","jurisdiction":"BVI","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"GRUPPO NUMMARIA LTD.","jurisdiction_description":"British Virgin Islands","service_provider":"Mossack Fonseca","incorporation_date":"25-NOV-1993","countries":"United Kingdom","country_codes":"GBR","ibcRUC":"101294","address":"AUSKERRY INVESTMENTS LIMITED 1, PARK ROAD LONDON NW1 6XN ENGLAND","status":"Defaulted","node_id":"10060886","jurisdiction":"BVI","sourceID":"Panama Papers"}},{"node_properties":{"valid_until":"The Panama Papers data is current through 2015","name":"F.S.C. LTD.-MARIAH OVERSEAS LIMITED","jurisdiction_description":"British Virgin Islands","service_provider":"Mossack Fonseca","incorporation_date":"04-JAN-1994","countries":"Switzerland","country_codes":"CHE","ibcRUC":"103935","address":"GOTTHARDSTRASSE 57 6045 MEGGEN SWITZERLAND","status":"Active","node_id":"10064221","jurisdiction":"BVI","sourceID":"Panama Papers"}}],"rows":10,"status":"200"}}

You will need to do the same for each of the other endpoints with its corresponding fetch function.

Your file should look like this at the end::

    from rest_framework.views import APIView
    from rest_framework.response import Response
    from .models.helpers import (
        count_nodes,
        fetch_nodes,
        fetch_node_details,
        fetch_countries,
        fetch_jurisdictions,
        fetch_data_source,
    )

    class GetNodesCount(APIView):
        def get(self, request):
            count_info = {
                'node_type': request.GET.get('t', 'Entity'),
                'name': request.GET.get('q', ''),
                'country': request.GET.get('c', ''),
                'jurisdiction': request.GET.get('j', ''),
                'sourceID': request.GET.get('s', ''),
            }
            count = count_nodes(count_info)
            data = {
                'response': {
                    'status': '200',
                    'data': count,
                },
            }
            return Response(data)

    class GetNodesData(APIView):
        def get(self, request):
            fetch_info = {
                'node_type': request.GET.get('t', 'Entity'),
                'name': request.GET.get('q', ''),
                'country': request.GET.get('c', ''),
                'jurisdiction': request.GET.get('j', ''),
                'sourceID': request.GET.get('s', ''),
                'limit': 10,
                'skip': int(request.GET.get('p', 1)),
            }
            nodes = fetch_nodes(fetch_info)
            data = {
                'response': {
                    'status': '200',
                    'rows': len(nodes),
                    'data': nodes,
                },
            }
            return Response(data)

    class GetNodeData(APIView):
        def get(self, request):
            node_info = {
                'node_type': request.GET.get('t', 'Entity'),
                'node_id': int(request.GET.get('id')),
            }
            node_details = fetch_node_details(node_info)
            data = {
                'response': {
                    'status': '200',
                    'data': node_details,
                },
            }
            return Response(data)

    class GetCountries(APIView):
        def get(self, request):
            countries = fetch_countries()
            data = {
                'response': {
                    'status': '200',
                    'data': countries,
                },
            }
            return Response(data)

    class GetJurisdictions(APIView):
        def get(self, request):
            jurisdictions = fetch_jurisdictions()
            data = {
                'response': {
                    'status': '200',
                    'data': jurisdictions,
                },
            }
            return Response(data)

    class GetDataSource(APIView):
        def get(self, request):
            data_source = fetch_data_source()
            data = {
                'response': {
                    'status': '200',
                    'data': data_source,
                },
            }
            return Response(data)
