# LLM, JEV, and CDMs: A Developer-Oriented Technical Guide

**Audience:** AI engineers, backend developers, and solution architects\
**Example domain:** Procure-to-Pay (P2P), including requirements,
vendors, RFQs, and quotation processing\
**Status:** Conceptual architecture and illustrative code.
Provider-specific APIs must be checked against current vendor
documentation.

## 1. Executive summary

These terms describe different things and should not be treated as
interchangeable model families:

-   **LLM (Large Language Model):** a general-purpose model that
    generates and interprets language and can perform many tasks.
-   **JEV (TypeSafe AI's product name):** a decision-oriented system
    described by its provider as accepting context and typed questions
    and returning structured decisions. "JEV" should be treated as a
    product name, not expanded into an assumed acronym.
-   **CDM (as used in the referenced article):** ScaleDown's term for
    specialized models for bounded tasks such as classification,
    extraction, summarization, and context compression. This is
    provider-specific terminology, not a universally standardized model
    category.
-   **Constrained decoding:** a separate technique that restricts the
    outputs an underlying generative model may produce, for example to
    valid JSON or a grammar. Do not assume that every use of "CDM" means
    this technique.
-   **Common Data Model (CDM):** another meaning of CDM in enterprise
    software; it is a standardized data schema, not the ScaleDown model
    concept discussed here.

The key design principle is to route each task to the simplest component
that meets its accuracy, latency, security, and auditability
requirements. Use an LLM for flexible interpretation and generation;
evaluate decision-oriented systems for bounded choices; evaluate
specialized extraction/classification/compression models for repetitive
tasks. Keep policy enforcement, arithmetic, authorization, and approval
in deterministic application code.

## 2. Terminology and scope

This guide follows the terminology in the Medium article:

-   Article: [After JEV Now CDMs Will Expand The
    Paradigm](https://medium.com/starling-elevate-it-solution/after-jev-now-cdms-will-expand-the-paradigm-ac82baf48d4c)
-   Jev product overview: [TypeSafe AI ---
    Jev](https://www.typesafeai.org/jev)
-   ScaleDown integration overview: [Building long-running agents with
    ScaleDown and Deep
    Agents](https://www.langchain.com/blog/building-long-running-agents-with-scaledown-and-deep-agents)

Product features, context limits, pricing, benchmarks, SDK signatures,
and model availability change over time. Confirm current details with
the providers before building a production dependency. Performance and
savings claims in marketing or opinion articles should be treated as
hypotheses to validate on your own workload.

## 3. Conceptual comparison

  ----------------------------------------------------------------------------------------
  Dimension         LLM                    JEV /               CDM as used in the article
                                           decision-oriented   
                                           model               
  ----------------- ---------------------- ------------------- ---------------------------
  Primary purpose   General language       Structured          Specialized bounded tasks
                    understanding and      decisions from      such as extraction,
                    generation             context and typed   classification,
                                           questions           summarization, or context
                                                               compression

  Typical input     Prompt, conversation,  Context plus a      Task-specific input and
                    document, sometimes    defined set of      instructions
                    images/audio           questions or answer 
                                           types               

  Typical output    Text, code, JSON, tool Typed               Labels, extracted
                    call, explanation      answers/choices and spans/fields, summaries, or
                                           associated scores   compressed context
                                           or probabilities    depending on product/task
                                           where supported     

  Open-ended        Core strength          Not the primary     Not the primary purpose
  generation                               purpose             

  Classification    Yes                    A core fit          A core fit

  Source-grounded   Possible; needs        Not its primary     A stated use case for
  extraction        validation             distinguishing use  extraction-focused models

  Strict output     Use structured output  Typed result        Depends on model/API
  format            or constrained         interface,          contract
                    decoding if supported  depending on        
                                           product             

  Best fit          Complex, varied,       Repeated decisions  High-volume, bounded tasks
                    ambiguous work         from a defined      
                                           question space      

  Key risk          Hallucination, output  Scores may be       Extraction/classification
                    variance, latency/cost misinterpreted;     errors; task coverage may
                                           product-specific    be narrow
                                           limits              

  Integration role  Reasoning/generation   Decision component  Specialized processing
                    engine                                     component
  ----------------------------------------------------------------------------------------

**Important:** This table is a high-level selection guide, not a claim
that these systems share the same internal architecture or that one is
always more accurate or cheaper.

## 4. What happens inside an LLM?

A common LLM architecture uses a Transformer neural network. During
inference, the model processes the supplied context and generates output
token by token. Depending on the model and runtime, it may also support
tool calling, multimodal input, and structured output.

Typical workflow:

1.  Prepare trusted context (requirement document, vendor record, or
    email).
2.  State the task and output contract.
3.  Call the model.
4.  Parse and validate the response.
5.  Verify factual claims against source data.
6.  Apply business rules and route to a human when needed.

An LLM can return JSON, but "asked for JSON" is not the same as
"guaranteed schema-valid JSON." Use a provider's structured-output or
constrained-decoding feature when available, then validate the result in
application code.

## 5. What is JEV?

JEV is the product name used by TypeSafe AI. Its public positioning is
decision-oriented: provide context and typed questions, then receive
structured answers rather than asking a general-purpose model to compose
a long response.

Conceptual example:

-   Context: vendor quotation and requirement.
-   Question: "Does this vendor meet the required RAM?"
-   Allowed answer: `yes`, `no`, or `unknown`.
-   Application: validate the answer and retain the source evidence.

Do not infer hidden implementation details from this interface. The
following is conceptual pseudocode, not a verified SDK call:

``` python
# Conceptual pseudocode only — not a real provider SDK signature.
decision = jev_client.decide(
    context=quotation_text,
    questions=[
        {
            "name": "ram_match",
            "type": "enum",
            "choices": ["yes", "no", "unknown"],
        }
    ],
)

# Adapt parsing to the actual provider response schema.
ram_match = decision["ram_match"]
```

Before production use, verify the real SDK/API, accepted question types,
score semantics, limits, data retention, and error behavior in TypeSafe
AI's current documentation.

## 6. What is a CDM in the referenced article?

In the article's context, CDM refers to ScaleDown's specialized-model
approach for bounded operations such as classification, extraction,
summarization, and context compression. This is not a universal industry
definition of "CDM."

### 6.1 Classification

Choose one label from a known set.

Example labels for vendor emails:

-   `quotation`
-   `invoice`
-   `technical_question`
-   `delivery_update`
-   `other`

The output should be checked against the allowed labels. If the model
supports scores, do not treat them as calibrated probabilities unless
the provider documents that meaning and calibration has been tested.

### 6.2 Extraction

Return specific source-grounded values, ideally with provenance.

Example fields from a quotation:

-   vendor name
-   quotation number
-   currency and total price
-   unit price
-   delivery days
-   warranty period

Extraction should preserve uncertainty. If the document does not contain
a value, return `null` or `unknown` rather than inventing it. Validate
extracted numbers and dates against the original document.

### 6.3 Summarization and context compression

Reduce a large conversation or tool output to the information needed for
the next step. Compression is lossy by nature: important details can be
omitted. Preserve key facts, source identifiers, unresolved questions,
constraints, and decisions. For consequential actions, retrieve the
original source instead of relying only on a compressed summary.

## 7. Constrained decoding is not the same as a CDM product

**Constrained decoding** is a generation-time technique that restricts
output tokens according to a schema, grammar, or other constraints. For
example, it can help a generative model produce JSON that conforms to a
defined schema.

It does not independently make the output factually correct. A
syntactically valid object can still contain a wrong price or vendor
name.

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  CDM (ScaleDown usage in the         Provider-specific specialized-model
  article)                            terminology

  Constrained decoding                A technique for restricting a
                                      generative model's output

  Common Data Model                   A standardized data
                                      representation/schema in some
                                      enterprise ecosystems
  -----------------------------------------------------------------------

When someone says "CDM," ask which meaning they intend before choosing a
technology.

## 8. Runnable Python examples without proprietary SDKs

The following examples use only the Python standard library. They
demonstrate application-side contracts and validation. They do **not**
emulate an LLM, JEV, or ScaleDown model; replace the illustrative
classifier/extractor with a real model API when integrating a provider.

### 8.1 Define validated data structures

``` python
from dataclasses import dataclass
from typing import Literal

EmailCategory = Literal[
    "quotation",
    "invoice",
    "technical_question",
    "delivery_update",
    "other",
]

@dataclass
class VendorQuotation:
    vendor_name: str
    currency: str
    total_price: float
    delivery_days: int | None

def validate_category(value: str) -> EmailCategory:
    allowed = {
        "quotation",
        "invoice",
        "technical_question",
        "delivery_update",
        "other",
    }
    normalized = value.strip().lower()
    if normalized not in allowed:
        raise ValueError(f"Unsupported category: {value!r}")
    return normalized  # type: ignore[return-value]

def validate_quotation(data: dict) -> VendorQuotation:
    required = {"vendor_name", "currency", "total_price", "delivery_days"}
    missing = required - data.keys()
    if missing:
        raise ValueError(f"Missing fields: {sorted(missing)}")

    vendor_name = str(data["vendor_name"]).strip()
    currency = str(data["currency"]).strip().upper()
    total_price = float(data["total_price"])
    delivery_days = data["delivery_days"]

    if not vendor_name:
        raise ValueError("vendor_name cannot be empty")
    if len(currency) != 3 or not currency.isalpha():
        raise ValueError("currency must be a 3-letter code, e.g. INR")
    if total_price < 0:
        raise ValueError("total_price cannot be negative")
    if delivery_days is not None:
        delivery_days = int(delivery_days)
        if delivery_days < 0:
            raise ValueError("delivery_days cannot be negative")

    return VendorQuotation(
        vendor_name=vendor_name,
        currency=currency,
        total_price=total_price,
        delivery_days=delivery_days,
    )
```

### 8.2 A deterministic baseline classifier

This is a baseline for tests, not an AI model. It helps establish
labels, expected behavior, and an evaluation set before replacing it
with a model.

``` python
def baseline_email_classifier(text: str) -> EmailCategory:
    content = text.casefold()

    if any(word in content for word in ("invoice", "tax invoice", "bill number")):
        return "invoice"
    if any(word in content for word in ("quotation", "quote", "price offer")):
        return "quotation"
    if any(word in content for word in ("technical issue", "specification", "compatibility")):
        return "technical_question"
    if any(word in content for word in ("shipment", "delivery", "dispatch")):
        return "delivery_update"
    return "other"

if __name__ == "__main__":
    sample = "Please find attached our quotation and delivery schedule."
    print(baseline_email_classifier(sample))  # quotation
```

Keyword rules can misclassify ambiguous messages. In production, compare
this baseline with a trained classifier, JEV, or a CDM using a labeled
test set.

### 8.3 Validate extracted quotation data

``` python
raw_model_output = {
    "vendor_name": "PixelPro Technologies Pvt. Ltd.",
    "currency": "INR",
    "total_price": 85000,
    "delivery_days": 12,
}

quotation = validate_quotation(raw_model_output)

print(quotation)
# VendorQuotation(
#   vendor_name='PixelPro Technologies Pvt. Ltd.',
#   currency='INR',
#   total_price=85000.0,
#   delivery_days=12
# )
```

This verifies field types and simple business constraints. It does not
prove that the extracted data matches the source document. Add
provenance such as page number, paragraph, quoted source text, and
document ID, then compare high-impact values with the source.

### 8.4 Use deterministic code for procurement policy

``` python
def meets_delivery_deadline(
    delivery_days: int | None,
    max_allowed_days: int,
) -> bool | None:
    """True = meets deadline, False = misses it, None = unknown."""
    if delivery_days is None:
        return None
    return delivery_days <= max_allowed_days

print(meets_delivery_deadline(12, 15))  # True
print(meets_delivery_deadline(20, 15))  # False
print(meets_delivery_deadline(None, 15))  # None
```

A model may extract `delivery_days`; ordinary code should compare it
with the required deadline. Do not ask an LLM to perform a simple policy
check when a deterministic comparison is sufficient.

## 9. Provider integration pattern

Use a small adapter so your application does not depend on a vendor's
response format throughout the codebase.

``` python
from typing import Protocol, Any

class Classifier(Protocol):
    def classify(self, text: str) -> str:
        """Return a candidate category from a configured label set."""
        ...

class Extractor(Protocol):
    def extract(self, text: str) -> dict[str, Any]:
        """Return candidate fields extracted from source text."""
        ...

def process_vendor_email(
    text: str,
    classifier: Classifier,
    extractor: Extractor,
) -> dict[str, Any]:
    category = validate_category(classifier.classify(text))

    result: dict[str, Any] = {"category": category}

    if category == "quotation":
        fields = extractor.extract(text)
        quotation = validate_quotation(fields)
        result["quotation"] = quotation.__dict__

    return result
```

Implement `Classifier` and `Extractor` adapters using the actual SDK/API
for the chosen provider. Add timeouts, retries with limits, structured
logging, request IDs, redaction, and provider error handling. Avoid
logging full supplier documents or confidential prices unless policy
permits it.

## 10. Example P2P architecture

``` text
Requirement / Vendor Document / Email
                 |
                 v
        Document and access checks
                 |
                 v
              Task router
          /        |         \
         v         v          v
   Extraction   Classification  LLM
      CDM        JEV/CDM       reasoning
         \         |          /
          \        |         /
           v       v        v
         Typed result + source evidence
                    |
                    v
       Schema and business-rule validation
                    |
                    v
       Vendor comparison / RFQ workflow
                    |
                    v
          Human review and approval
                    |
                    v
         Auditable action and history
```

This is a proposed pattern. Run independent tasks in parallel only when
they do not depend on each other's outputs. Do not send confidential
data to a provider until the organization's data-processing requirements
have been reviewed.

## 11. Evaluation plan

Do not choose based on a headline benchmark alone. Build a
representative, labeled dataset from your own workflow and compare
alternatives.

  -----------------------------------------------------------------------
  Measure                             How to measure it
  ----------------------------------- -----------------------------------
  Classification quality              Per-class precision, recall, F1,
                                      confusion matrix

  Extraction quality                  Exact match / normalized match for
                                      each field; date and currency error
                                      rate

  Evidence quality                    Percentage of extracted values with
                                      correct source locations

  Decision quality                    Accuracy against expert-reviewed
                                      labels; calibration if scores are
                                      used

  Latency                             p50 and p95 end-to-end response
                                      time

  Cost                                Total cost per document/task,
                                      including retries and fallbacks

  Robustness                          Missing fields, malformed files,
                                      ambiguous wording, adversarial or
                                      irrelevant content

  Safety                              Unauthorized actions prevented,
                                      sensitive-data handling, approval
                                      compliance
  -----------------------------------------------------------------------

Include an `unknown` or escalation path. Measure false acceptance
separately for high-impact procurement fields such as price, delivery
commitment, and supplier identity.

A practical rollout: 1. Create a human-labeled test set. 2. Establish a
deterministic baseline. 3. Test the LLM and any available JEV/CDM
product on the same cases. 4. Compare accuracy, p95 latency, and total
cost. 5. Deploy in shadow mode and compare results with human decisions.
6. Add a bounded fallback path and monitor drift. 7. Require human
approval for consequential procurement actions.

## 12. Security and production checklist

-   Validate all model outputs at the application boundary.
-   Treat documents and email content as untrusted input; they may
    contain prompt-injection instructions.
-   Separate model suggestions from tool permissions.
-   Apply least-privilege access to files, email, vendors, and purchase
    workflows.
-   Never let a model bypass approval, spending limits, or authorization
    checks.
-   Record model/provider version, prompt/template version, source
    document ID, validation result, and reviewer decision.
-   Redact secrets and personal data from logs.
-   Set timeouts, bounded retries, rate limits, and graceful fallbacks.
-   Test malformed, incomplete, conflicting, and malicious inputs.
-   Verify provider retention, regional processing, and contractual
    terms.
-   Re-evaluate when models, prompts, documents, or business policies
    change.

## 13. Decision guide

  -----------------------------------------------------------------------
  Requirement                         Starting point
  ----------------------------------- -----------------------------------
  Explain a complex requirement or    LLM
  write an RFQ email                  

  Select one answer from typed        Evaluate JEV or a suitable
  questions                           classifier

  Extract exact fields from many      Evaluate an extraction CDM and/or a
  quotations                          document extraction model

  Categorize high-volume emails       Evaluate a classification CDM, JEV,
                                      or conventional classifier

  Shorten long agent/tool context     Evaluate context
                                      summarization/compression

  Ensure JSON/schema correctness      Structured outputs or constrained
                                      decoding, plus validation

  Compare prices and enforce delivery Deterministic code
  thresholds                          

  Approve a purchase or send a        Authorization rules and human
  consequential action                approval
  -----------------------------------------------------------------------

## 14. References

1.  Puneet Singhal, *After JEV Now CDMs Will Expand The Paradigm*:
    https://medium.com/starling-elevate-it-solution/after-jev-now-cdms-will-expand-the-paradigm-ac82baf48d4c
2.  TypeSafe AI, *Jev*: https://www.typesafeai.org/jev
3.  LangChain, *Building long-running agents with ScaleDown and Deep
    Agents*:
    https://www.langchain.com/blog/building-long-running-agents-with-scaledown-and-deep-agents

**Final takeaway:** LLM, JEV, and CDM are not direct substitutes in
every situation. Use general-purpose generation for open-ended tasks,
decision-oriented models for typed decisions, and specialized models for
bounded operations---while keeping validation, business rules, security,
and approvals in ordinary application code.
