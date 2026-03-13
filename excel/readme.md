# Naming Tags in Rater Excel
#### Naming Rules
*   Do use ONLY lower case letters in naming. Do NOT use upper case letters.
*   Do use `_` (underscore) as word separators
*   ~~Do NOT use~~ `.` ~~(dot) as part of name (unless its an object member)~~
*   Do NOT use `user`/`broker` properties as cell names
*   Do NOT use reserved names (ex: `product_name`)
*   Do NOT use value as cell name ("`pls_provide_an_indicative_amount_and_cause_of_Loss`")
*   Do NOT start with or end with spaces/tabs
*   **Arrays**
    *   Do keep the array cells all empty
    *   Do name all template cells with name ending with `_tmpl`
    *   Do NOT use array name on another cell (ex: `arr_locations` and `locations`)
    *   Do NOT use array cell names on another cell (ex: `sum_insured_tmpl` and `sum_insured`)
    *   Do NOT use long descriptive names as part of large array templates
    *   Do prefix name with part array name if it clashes with common names (ex: `location_sum_insured_tmpl`)
    *   Do use numbers only to represent an Array of values (that require looping thru)
*   Some examples
    *   **Cell Name**

| | **Correct naming** | **Wrong naming** |
| --- | --- | --- |
| Remove prepositions such as "of", “and”, “or “, and “the" from names unless it is critical to make sense of the name.<br><br>An object name SHOULD only contain verbs, nouns, and adjectives unless it adds clarity to the name | `registration_date`<br>`as_of_date` | `date_of_registration` (proposition not critical)<br>`date_of_reg` (proposition not critical)<br>`as_date` (unclear name) |
| Names MUST be in singular form unless the concept itself is plural | `item_count`<br>`goods_quantity` | `items_count`<br>`good_quantity` |

*   Do NOT use acronyms, abbreviations, or other word truncations, except those mentioned in **Section C** below.
*   Do have separate sheet in Product XLS called “Naming” with below structure for all the inputs and output names used:
*   **Cell Name**

| | **Name Category** | **Description** |
| --- | --- | --- |
| proposer_dob | proposer | Proposer Date of Birth |

*   All names to start with a valid Insillion Name Category followed by name intended for the cell. Refer Section **[B]** for the Name Categories

# Tags - to avoid
#### Reserved Names
Do NOT use below names / tags in any of your Products as these are reserved names used by Insillion internally and can be overwritten or have unpredictable behaviour.

| **Category** | **Reserved Names** |
| --- | --- |
| User | agent_broker_id, agent_code, assigned_to, assigned_by, author, broker_id, broker_code, broker_group_id, group_id, c_ts, created_by, ip, u_ts, uuid |
| Contact info | agent_email, agent_first_name, agent_last_name, agent_name, broker_address_1, broker_name, broker_phone_no_1, email, first_name, last_name, mobile_no |
| Product | par_product_id, product_id, product_name, product_group_id, role_id, wf_id, wf_name, rule_id, rule_name |
| Transaction | document_id, iagree_id, payment_id, policy_id, policy_no, policy_status_id, proposal_id, proposal_no, quote_id, quote_no, stage, stage_index, verify_id, par_quote_id, parent_policy_no, customer_id, cert_id, master_policy_no, combo_policy_id, renewal_count, col_no |
| Referral | qnstp_case, qnstp_enabled, qnstp_id, nstp_details_id, nstp_id, nstp_enabled, nstp_status, pnstp_enabled, pnstp_id |
| Amount | agent_deposit_balance |
| Other | dashboard_id, menu_id, plugin_id, query_id, widget_id, status, cust_fld_1...n, otp_verified, agreed_by, ready_state, status_desc |
| Transactions Dates | valid_till, quote_date, pdf_date, publish_date, expiry_date, agreed_on, issue_date, policy_start_date_dt, policy_end_date_dt, policy_start_date_dt_ext, policy_end_date_dt_ext |


# Tags - to use
This page provides a list of commonly used tags by Insillion across all products.

**Tags - Common** 
These are tags that can be used for reporting and aggregation. It is encouraged to use the below names as Insillion recognizes these tags internally and attaches a special meaning based on the context.

| **Common Name** | **Context of when to use the Common name** |
| --- | --- |
| `premium_value` | Use this for the cell that has the total premium without tax |
| `sum_insured` | Use this for the cell that has Total sum insured at policy level |
| `first_name` | Use this to store the Policy holder's first name |
| `last_name` | Use this to store the Policy holder's last name |
| `locations` | Use this for list of locations (i.e. `arr_locations`) |

**Tags - Computed**
Below tag names can be affected by Insillion's platform based on the context

| **Common Name** | **Computation done** |
| --- | --- |
| `policy_start_date` | Use this for Risk start date.<br>Insillion automatically moves this to the date of payment received, only if the LOB -> Payment -> "Adjust Effective Date" is selected. |
| `policy_end_date` | Use this for Risk end date.<br>Insillion automatically moves this to the date of payment received + policy tenure, only if the LOB -> Payment -> "Adjust Effective Date" is selected. |
| `prev_policy_start_date` | Use this to store the Previous Policy's Start Date |
| `prev_policy_end_date` | Use this to store the Previous Policy's End Date |
| `payment.total_rcvd` | Insillion stores the total payment received against a Policy based on the Payment plugin used |
| `payment.total_amt` | Insillion stores the total demand against a Policy, in this tag |
| `hybrid_premium` | Pro-rated Premium in the case of Endorsements (considering even out of sequence endorsements) |
