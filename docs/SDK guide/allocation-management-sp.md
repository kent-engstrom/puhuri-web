# Allocation Lifecycle Management by Service Provider

This page describes operations to be performed by service provider.

## Prerequisites

Please, read [initial setup for Puhuri Core SDK](initial-setup.md) and
please reach out to [support email](mailto:support@hpc.ut.ee) to get credentials for Puhuri Core.


## Getting a list of users

`list_users` method is used to fetch all users in Puhuri Core.

```python
result = client.list_users()

# result =>[
# {'affiliations': [],
#   'agreement_date': None,
#   'civil_number': None,
#   'competence': '',
#   'customer_permissions': [],
#   'date_joined': '2021-06-23T07:59:59.511681Z',
#   'description': '',
#   'email': 'arto.tuomi@csc.fi',
#   'first_name': 'PUHURI',
#   'full_name': 'PUHURI Test User 100',
#   'is_active': True,
#   'is_staff': False,
#   'is_support': False,
#   'job_title': '',
#   'last_name': 'Test User 100',
#   'native_name': '',
#   'organization': '',
#   'phone_number': '',
#   'preferred_language': '',
#   'project_permissions': [{'customer_name': 'IT CENTER FOR SCIENCE LTD',
#                            'pk': 9,
#                            'project_name': 'Project from CSC',
#                            'project_uuid': 'c3be1c35f36448068cb57e278b788318',
#                            'role': 'manager',
#                            'url': 'https://puhuri-core-demo.neic.no/api/project-permissions/9/'}],
#   'registration_method': 'eduteams',
#   'requested_email': None,
#   'url': 'https://puhuri-core-demo.neic.no/api/users/f8347a45f2f4409ea6be2034edc5e91b/',
#   'username': '01cfb7d6b76d400d12b8c8e0e33e36c5ef4562c1@acc.researcher-access.org',
#   'uuid': 'f8347a45f2f4409ea6be2034edc5e91b'}]
```

## Getting a list of SSH keys

`list_ssh_keys` method is used to fetch all SSH keys in Puhuri Core.

```python
result = client.list_ssh_keys()

# result =>
[{'fingerprint': '...',
  'is_shared': False,
  'name': 'new key',
  'public_key': '... '
                '...'
                '...',
  'type': 'ssh-ed25519',
  'url': '...',
  'user_uuid': '31165878c7a04a4197f282924c1a2418',
  'uuid': '8ae5c9bdbafd44c78250bc98c9783fad'},
```

## Getting a list of resource allocations

`list_marketplace_resources` method is used to fetch resources related to offerings, which belong to user's service provider.

Possible filter options for allocations (each one is optional):

- `provider_uuid` - UUID of a service provider organization;
- `state` - current state of a resource allocation; valid values: `Creating`, `OK`, `Erred`, `Updating`, `Terminating`, `Terminated`;
- `offering_uuid` - UUID of a related resource;
- `fields` - list of fields to return.

```python
result = client.list_marketplace_resources(
    provider_uuid='<provider-uuid>',
    state='Creating',
    offering_uuid='<offering-uuid>',
    fields=['name', 'offering', 'state', 'limits', 'plan', 'project', 'url']
)

# result => {
#   'limits': {'cpu_k_hours': 1000, 'gb_k_hours': 3000, 'gpu_k_hours': 1000},
#   'name': 'Sample resource',
#   'offering': 'https://puhuri-core-demo.neic.no/api/marketplace-offerings/cf4eb9c29fc74af4ade667fcb53633d5/',
#   'plan': 'https://puhuri-core-demo.neic.no/api/marketplace-plans/1537de6e94f9427cafb74cb63fa21c72/',
#   'project': 'https://puhuri-core-demo.neic.no/api/projects/8a158ebf1abf4c74a431b9c65a0d7829/',
#   'state': 'Creating',
#   'url': 'https://puhuri-core-demo.neic.no/api/marketplace-resources/a1916bd53fd04b1ab1a4e700c926607b/'
# }
```

## Approving/rejecting order items for allocations

A consumer can request allocation creation, update or termination using `OrderItem` entity.
The service provider can either approve or reject it using
`marketplace_order_item_approve` and `marketplace_order_item_reject` correspondingly.
Both methods obtain **`order_item_uuid`** - UUID of an order item for the allocation.
Only users with service provider owner and manager roles can perform these actions.

For example, a consumer requested an allocation using `OrderItem` with `CREATE` type. After that, an empty allocation with `CREATING` state was appeared.
A service provider can change the state to `OK` (created successfully) using `marketplace_order_item_approve` or
`Terminated` (creation rejected) using `marketplace_order_item_reject`.
In order to get a proper order item, SP owner can use `list_order_items` method. This action is for order item listing and supports filtering by state and allocation.

```python
from waldur_client import WaldurClient

order_items = client.list_order_items({'state': 'executing', 'resource_uuid': '<allocation-uuid>'})

order_item = order_items[0]

result = client.marketplace_order_item_approve(
    resource_uuid=order_item['uuid'],
)

# result => {
#   'details': 'Order item has been approved.'
#}
```

## Updating resource allocation with local reference (setting `backend_id` field)

Each allocation can have a link to a service provider's internal reference using `backend_id` field. Only users with service provider owner and manager roles can set this value using `marketplace_resource_set_backend_id` method of the client. It requires the following arguments:

- **`resource_uuid`** - UUID of a resource allocation;
- **`backend_id`** - id of an external entity.

```python
result = client.marketplace_resource_set_backend_id(
    resource_uuid='<resource-uuid>',
    backend_id='<some-backend-id>'
)

# result => {
#   'status': 'Resource backend_id has been changed.'
# }
```

In case if SDK usage is not possible, HTTP request can be sent:

```http
POST <API-URL>/marketplace-resources/<resource-uuid>/

{
    "backend_id": "<some-backend-id>"
}

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

{
    "status": "Resource backend_id has been changed."
}
```

## Providing additional access detail for resource allocation

For additional details related to allocation access, `report` field is used.
In order to provide this information, owners and managers can use `marketplace_resource_submit_report` method. It requires the following arguments:

- **`resource_uuid`** - UUID of a resource allocation;
- **`report`** - list of `ResourceReportRecord` instances.

```python
from waldur_client import WaldurClient, ResourceReportRecord

result = client.marketplace_resource_submit_report(
    resource_uuid='<resource-uuid>',
    report=[
        ResourceReportRecord(header='Header1', body='Body1'),
        ResourceReportRecord(header='Header2', body='Body2')
    ]
)

# result => {
#   'status': 'Report is submitted'
# }
```

## Getting a list of members in a project with active resource allocations

Service provider owners and managers can list project members using a resource allocation with `marketplace_resource_get_team` method.
It requires the following arguments:

- **`resource_uuid`** - UUID of a resource allocation.

```python
result = client.marketplace_resource_get_team(
    resource_uuid='<resource-uuid>'
)

# result => {
# [
#  {'email': 'CSC_api_user@example.com',
#   'expiration_time': None,
#   'full_name': 'Integration user (CSC)',
#   'permission': 'https://puhuri-core-demo.neic.no/api/project-permissions/10/',
#   'role': 'admin',
#   'url': 'https://puhuri-core-demo.neic.no/api/users/bb235c67dcb44470a45d4e0f94e0ed00/',
#   'username': 'CSC_api_user',
#   'uuid': 'bb235c67dcb44470a45d4e0f94e0ed00'},
#  {'email': 'ETAIS_api_user@example.com',
#   'expiration_time': None,
#   'full_name': 'Integration user (ETAIS)',
#   'permission': 'https://puhuri-core-demo.neic.no/api/project-permissions/3/',
#   'role': 'member',
#   'url': 'https://puhuri-core-demo.neic.no/api/users/0cc72de4ebb840d98fef133e4433ec9a/',
#   'username': 'ETAIS_api_user',
#   'uuid': '0cc72de4ebb840d98fef133e4433ec9a'}]
# }
```

## Reporting usage for a resource allocation

A usage of a resource allocation can be submitted by a corresponding service provider.
For this, the following methods are used:

- `get_marketplace_offering` - getting offering with components info.
  Arguments:

    - **`offering_uuid`** - UUID of an offering

- `marketplace_resource_get_plan_periods` - getting current plan periods for resource allocation. Arguments:

    - **`resource_uuid`** - UUID of a resource

- `create_component_usages` - creating or updating components usage for the current plan

    - **`plan_period_uuid`** - UUID of a plan period
    - **`usages`** - list of `ComponentUsage` instances

```python
from waldur_client import WaldurClient, ComponentUsage

offering = client.get_marketplace_offering(offering_uuid='<offering-uuid>')
component_types = [component['type'] for component in offering['components']]
plan_periods = client.marketplace_resource_get_plan_periods(resource_uuid='<resource-uuid>')

if len(plan_periods) > 0:
    plan_period = plan_periods[0]

    client.create_component_usages(
        plan_period_uuid=plan_period['uuid'],
        usages=[ComponentUsage(component_type=comp_type, amount=10, description='Usage') for comp_type in component_types]
    )

result = client.marketplace_resource_get_plan_periods('<resource-uuid>')

# result => {
# [{'components': [{'created': '2021-08-11T15:36:45.562440Z',
#                   'date': '2021-08-11T15:37:30.556830Z',
#                   'description': 'Usage',
#                   'measured_unit': 'units',
#                   'name': 'CPU',
#                   'type': 'cpu',
#                   'usage': 10,
#                   'uuid': '080066bebf9547d5ad3279f7c353b023'},
#                  {'created': '2021-08-11T15:36:45.842385Z',
#                   'date': '2021-08-11T15:37:30.556830Z',
#                   'description': 'Usage',
#                   'measured_unit': 'units',
#                   'name': 'GPU',
#                   'type': 'gpu',
#                   'usage': 10,
#                   'uuid': 'e8d745049bff4410b60f517e56daf421'}],
#   'end': None,
#   'plan_name': 'sample-plan',
#   'plan_uuid': '9f7b1d113f464214bb195db38c217cf9',
#   'start': '2021-08-11T15:20:25.762775Z',
#   'uuid': 'fb36ce56e53b43e59b3d963354d0467c'}]
# }

```

## Granting user access to resource

An access to a resource can be granted by service provider for a particular user.
For this purpose, `create_remote_offering_user` should be invoked. This method requires the following arguments:

- **`offering`** - UUID or URL of a target offering;
- **`user`** - UUID or URL of a target user;
- `username` - username for the user.

```python
result = client.create_remote_offering_user(
    offering='<offering-uuid-or-url>',
    user='<user-uuid-or-url>',
    username='<username>'
)

# result => {
#  'created': '2021-08-12T15:22:18.993586Z',
#  'offering': 'http://localhost:8000/api/marketplace-offerings/d47ca5bce71144579df29da3c290027e/',
#  'offering_name': 'Remote offering (really)',
#  'offering_uuid': 'd47ca5bce71144579df29da3c290027e',
#  'user': 'http://localhost:8000/api/users/db157a5cf7f247eba161cd90eba9ac63/',
#  'user_uuid': 'db157a5cf7f247eba161cd90eba9ac63',
#  'username': 'abc'
# }
```

In case if SDK usage is not possible, HTTP request can be sent:

```http
POST <API-URL>/marketplace-offering-users/

{
    "offering": "<offering-uuid-or-url>",
    "user": "<user-uuid-or-url>",
    "username": "<username>"
}

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

{
    "created": "2021-08-12T15:22:18.993586Z",
    "offering": "http://localhost:8000/api/marketplace-offerings/d47ca5bce71144579df29da3c290027e/",
    "offering_name": "Remote offering (really)",
    "offering_uuid": "d47ca5bce71144579df29da3c290027e",
    "user": "http://localhost:8000/api/users/db157a5cf7f247eba161cd90eba9ac63/",
    "user_uuid": "db157a5cf7f247eba161cd90eba9ac63",
    "username": "abc"
}
```