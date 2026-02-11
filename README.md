TechShop | Automated Returns (n8n)

What is this project?
This project automates customer product return requests.
Instead of handling returns manually, everything is processed automatically using n8n.

A customer submits a return request using a simple form.
The request is sent to an n8n workflow.
The workflow checks the data, applies rules, saves the return, and notifies the team.

----

How it works

1. A user fills a return form (order id, email, product, reason).
2. The form sends the data to n8n through a webhook.
3. n8n checks that all required fields are present.
4. n8n checks the security token.
5. n8n looks for the order in the Orders Google Sheet.
6. n8n looks for the product in the Products Google Sheet.
7. n8n checks if a return already exists (anti-duplicate).
8. n8n decides if the return is accepted or refused.
9. n8n calculates the refund amount.
10. n8n saves the return in the Returns Google Sheet.
11. n8n writes a log entry in the Logs Google Sheet.
12. n8n sends a Slack message to the correct channel.
13. n8n returns an HTTP response to the client.

----

Webhook endpoint

POST /webhook/returns/intake

Required header:
x-techshop-token = my_secret_token

The workflow must be ACTIVE for this endpoint to work.

You can find my google sheet file shared here : https://docs.google.com/spreadsheets/d/1JgFE5DXQSSWsJDUX27WBdig2w_LsSeQWS1PUIVAtyoo/edit?usp=sharing

----

Request example

request_id: unique request id
created_at: ISO date
channel: contact_form
order_id: ORD-2025-001
customer_email: john@example.com
sku: PROD-001
reason: Défectueux
product_condition: Bon

----

Eligibility rules

A return is accepted if:

- The order is less than 30 days old
- The reason is valid:
  Défectueux
  Trop petit
  Trop grand
  Erreur commande
  Changement d'avis

Refund rules:

- Comme neuf = 100%
- Bon = 80%
- Endommagé = 50%

Warranty rule:
If the product has warranty AND
the reason is "Défectueux" AND
the product is "Endommagé"
→ refund is 100%

----

Anti-duplicate rule

dedupe*key = order_id + "*" + sku

If a return already exists with this key and status:
PENDING or APPROVED
→ the request is refused with HTTP 409.

----

Return ID

Each return gets a unique ID:
RET-YYYYMMDD-XXXX

Example:
RET-20260210-A7F2

----

Notifications

If return is accepted:
Slack message sent to #logistique

If return is refused:
Slack message sent to #support-general

Slack errors do not block the workflow.

----

API responses

201 : return created
404 : order not found or product not found
409 : duplicate return
401 : invalid token

----

Data storage (Google Sheets)

Orders : customer orders
Products : product information
Returns : return requests
Logs : workflow logs

----

Tools used

- n8n (local)
- Google Sheets
- Slack
- HTML + JavaScript  TO CREATE A FORM

----

Goal of this project

Reduce manual work.
Avoid duplicate returns.
Have clear rules.
Keep everything simple and traceable.
