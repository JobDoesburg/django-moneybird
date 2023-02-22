# Django Moneybird

### Usage

Put the following settings in your `settings.py` file:

```python
import os
from moneybird.webhooks.events import WebhookEvent

MONEYBIRD_ADMINISTRATION_ID = int(os.environ.get("MONEYBIRD_ADMINISTRATION_ID", 0))
MONEYBIRD_API_KEY = os.environ.get("MONEYBIRD_API_KEY")
MONEYBIRD_RESOURCE_TYPES = [
    "myapp.moneybird_resources.ContactResourceType",
]

# If you want to receive webhook events, you should define the following:
MONEYBIRD_WEBHOOK_EVENTS = [
    WebhookEvent.CONTACT,
]
MONEYBIRD_WEBHOOK_ID = os.environ.get("MONEYBIRD_WEBHOOK_ID")
MONEYBIRD_WEBHOOK_TOKEN = os.environ.get("MONEYBIRD_WEBHOOK_TOKEN")
MONEYBIRD_WEBHOOK_SITE_DOMAIN = os.environ.get("MONEYBIRD_WEBHOOK_SITE_DOMAIN") # e.g. "https://example.com"
MONEYBIRD_WEBHOOK_ALLOW_INSECURE = False # Set to False for testing purposes only

MONEYBIRD_AUTO_PUSH = True # Push changes to Moneybird automatically (so you don't have to call `instance.push_to_moneybird()` manually)
MONEYBIRD_FETCH_BEFORE_PUSH = False # Fetch the latest data from Moneybird before pushing changes. This is useful if you want to avoid overwriting changes made in Moneybird, but it will slow down your application. With webhooks, this is likely not necessary.
```

Your models should inherit from `SynchronizableMoneybirdResourceModel` (if they have a synchronization endpoint, see developer.moneybird.com) or `MoneybirdResourceModel`:

```python
from django.db import models
from django.utils.translation import gettext_lazy as _

from moneybird.models import SynchronizableMoneybirdResourceModel

class Contact(SynchronizableMoneybirdResourceModel):
    company_name = models.CharField(
        verbose_name=_("company name"),
        max_length=255,
        null=True,
        blank=True,
    )
    first_name = models.CharField(
        verbose_name=_("first name"),
        max_length=255,
        null=True,
        blank=True,
    )
    last_name = models.CharField(
        verbose_name=_("last name"),
        max_length=255,
        null=True,
        blank=True,
    )
```


Finally, define a `MoneybirdResourceType` for each model, like this:

```python
from moneybird.resources import ContactResourceType

from myapp.models import Contact

class ContactResourceType(ContactResourceType):
    model = Contact

    @classmethod
    def get_model_kwargs(cls, resource_data):
        kwargs = super().get_model_kwargs(resource_data)
        kwargs["company_name"] = resource_data["company_name"]
        kwargs["first_name"] = resource_data["firstname"]
        kwargs["last_name"] = resource_data["lastname"]
        ...
        return kwargs
        
    @classmethod
    def serialize_for_moneybird(cls, instance):
        data = super().serialize_for_moneybird(instance)
        data["company_name"] = instance.company_name or ""
        data["firstname"] = instance.first_name or ""
        data["lastname"] = instance.last_name or ""
        ...
        return data

```

For document lines, your model should extend `MoneybirdDocumentLineModel`.


### Webhooks
If you want to use webhooks, you should add the following to your `urls.py`:

```python
from django.urls import include, path

urlpatterns = [
    ...,
    path("moneybird/", include("moneybird.webhooks.urls")),
    ...
]
```

### Syncing
To sync all resources, you can use the `moneybirdsync` management command or call `moneybird.synchronization.synchronize()`.