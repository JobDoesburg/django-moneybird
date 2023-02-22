# Django Moneybird

### Usage

Put the following settings in your `settings.py` file:

```python
MONEYBIRD_ADMINISTRATION_ID = int(os.environ.get("MONEYBIRD_ADMINISTRATION_ID", 0))
MONEYBIRD_API_KEY = os.environ.get("MONEYBIRD_API_KEY")
MONEYBIRD_RESOURCE_TYPES = [
    "accounting.models.document_style.DocumentStyleResourceType",
    "accounting.models.tax_rate.TaxRateResourceType",
    "accounting.models.workflow.WorkflowResourceType",
    "accounting.models.ledger_account.LedgerAccountResourceType",
    "accounting.models.product.ProductResourceType",
    "accounting.models.project.ProjectResourceType",
    "accounting.models.contact.ContactResourceType",
    "accounting.models.recurring_sales_invoice.RecurringSalesInvoiceResourceType",
    "accounting.models.estimate.EstimateResourceType",
    "accounting.models.sales_invoice.SalesInvoiceResourceType",
    "accounting.models.purchase_document.PurchaseInvoiceDocumentResourceType",
    "accounting.models.purchase_document.ReceiptResourceType",
    "accounting.models.general_journal_document.GeneralJournalDocumentResourceType",
    "accounting.models.subscription.SubscriptionResourceType",
]
MONEYBIRD_WEBHOOK_EVENTS = [
    WebhookEvent.CONTACT,
    WebhookEvent.SALES_INVOICE,
    WebhookEvent.DOCUMENT,
    WebhookEvent.ESTIMATE,
    WebhookEvent.RECURRING_SALES_INVOICE,
    WebhookEvent.SUBSCRIPTION_CANCELLED,
    WebhookEvent.SUBSCRIPTION_CREATED,
    WebhookEvent.SUBSCRIPTION_UPDATED,
    WebhookEvent.SUBSCRIPTION_EDITED,
    WebhookEvent.SUBSCRIPTION_DESTROYED,
    WebhookEvent.TAX_RATE_UPDATED,
    WebhookEvent.TAX_RATE_CREATED,
    WebhookEvent.TAX_RATE_DEACTIVATED,
    WebhookEvent.TAX_RATE_DESTROYED,
    WebhookEvent.TAX_RATE_ACTIVATED,
    WebhookEvent.WORKFLOW,
    WebhookEvent.LEDGER_ACCOUNT,
    WebhookEvent.PRODUCT_UPDATED,
    WebhookEvent.PRODUCT_CREATED,
    WebhookEvent.PRODUCT_DEACTIVATED,
    WebhookEvent.PRODUCT_DESTROYED,
    WebhookEvent.PRODUCT_ACTIVATED,
    WebhookEvent.PROJECT_UPDATED,
    WebhookEvent.PROJECT_CREATED,
    WebhookEvent.PROJECT_ARCHIVED,
    WebhookEvent.PROJECT_DESTROYED,
    WebhookEvent.PROJECT_ACTIVATED,
    WebhookEvent.DOCUMENT_STYLE_CREATED,
    WebhookEvent.DOCUMENT_STYLE_UPDATED,
    WebhookEvent.DOCUMENT_STYLE_DESTROYED,
]
MONEYBIRD_WEBHOOK_ID = os.environ.get("MONEYBIRD_WEBHOOK_ID")
MONEYBIRD_WEBHOOK_TOKEN = os.environ.get("MONEYBIRD_WEBHOOK_TOKEN")

MONEYBIRD_AUTO_PUSH = True
MONEYBIRD_FETCH_BEFORE_PUSH = False
```

Your models should inherit from `SynchronizableMoneybirdResourceModel` (if they have a synchronization endpoint, see developer.moneybird.com) or `MoneybirdResourceModel`:

```python
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