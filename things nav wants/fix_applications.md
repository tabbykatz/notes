# Fix for newly discovered failed migration on Applications

## Background

Upon creating a new admin page, it was dicovered that while the thouroughly tested tool worked locally it failed in production. The cause was a call to ApplicationRepository to get `user_id`. This failed because `user_id` was not a column on Applicaiton in production, though it is in developement. With the help of Anthony Ruozzi, I learned that a migration to add `user_id` from at least a year ago had failed in production without notice but appeared successful in development. The reason I was the first to discover this problem is my unconventional query, where the preferred query would be on UserApplication or PreApplication. These child tables of Application both have the requisite columns, and this is why a fix is not straightforward.

Upon discussion with Anthony, Sandeep, and Nav, it was decided that it is preferable to not have the `user_id` column and that instead of fixing the migration we'd remove the column from development.

## Solution

I've been asked to drop the column if it exists without dropping the child tables' `user_id` columns. This migration should only run in dev if possible.

## First Attempt

```ruby
class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    remove_column :applications, :user_id, :integer
  end
end
```

### Before running the above migration:

```ruby
[1] pry(main)> Application.column_names.include? 'user_id'
=> true
[2] pry(main)> UserApplication.column_names.include? 'user_id'
=> true
[3] pry(main)> PreApplication.column_names.include? 'user_id'
=> true
[4] pry(main)>
```

### Warning

```ruby
=== Dangerous operation detected #strong_migrations ===

Active Record caches attributes which causes problems
when removing columns. Be sure to ignore the column:

class Application < ApplicationRecord
  self.ignored_columns = ["user_id"]
end

Deploy the code, then wrap this step in a safety_assured { ... } block.

class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    safety_assured { remove_column :applications, :user_id, :integer }
  end
end
```

```ruby
class RemoveUserIdFromApplications < ActiveRecord::Migration[6.0]
  def change
    safety_assured { remove_column :applications, :user_id, :integer }
  end
end

```

### After running this migration

```ruby
[5] pry(main)> Application.column_names.include? 'user_id'
=> false
[6] pry(main)> UserApplication.column_names.include? 'user_id'
=> false
[7] pry(main)> PreApplication.column_names.include? 'user_id'
=> false
```

### Result: Failure

Dropping the column on the parent removed it from the children as well.

## Another migration

This migration is only different in that it checks to see if the column exists, so I do not expect it to fix the problem of the previous attempt.

```ruby
if ActiveRecord::Base.connection.column_exists?(:applications , :user_id)
  change_table(:applications) do |t|
    t.remove :user_id
end
```

## Findings

I have not been able to drop this column without altering the child tables. According to [postgreSQL documentation](https://www.postgresql.org/docs/current/ddl-inherit.html), this is expected behaviour and cannot be prevented if the tables inherit from one another.

> `ALTER TABLE` will propagate any changes in column data definitions and check constraints down the inheritance hierarchy.

I exec'ed into the postgresql Docker container locally, connecting to `db/monolith`, I wanted to see if Rails magic was causing the children to drop columns.

Applications:

```SQL
                                                                       Table "public.applications"
             Column              |            Type             | Collation | Nullable |                 Default                  | Storage  | Stats target | Description
---------------------------------+-----------------------------+-----------+----------+------------------------------------------+----------+--------------+-------------
 id                              | bigint                      |           | not null | nextval('applications_id_seq'::regclass) | plain    |              |
 user_account_id                 | bigint                      |           |          |                                          | plain    |              |
 current_step                    | character varying           |           |          |                                          | extended |              |
 fee                             | integer                     |           |          |                                          | plain    |              |
 min_raising                     | integer                     |           |          |                                          | plain    |              |
 raising_amount                  | numeric(19,4)               |           |          |                                          | main     |              |
 max_loan_amount_at_signup       | integer                     |           | not null |                                          | plain    |              |
 initial_requested               | numeric(19,4)               |           |          |                                          | main     |              |
 initial_doc_number              | integer                     |           |          |                                          | plain    |              |
 contract_type                   | character varying           |           |          |                                          | extended |              |
 loan_funding_method             | character varying           |           |          |                                          | extended |              |
 turndown_partner                | character varying           |           |          |                                          | extended |              |
 use_of_funds                    | character varying           |           |          |                                          | extended |              |
 in_school                       | boolean                     |           |          | false                                    | plain    |              |
 is_psl                          | boolean                     |           |          | false                                    | plain    |              |
 compare_opt_in                  | boolean                     |           |          |                                          | plain    |              |
 last_decision_period_start_date | date                        |           |          |                                          | plain    |              |
 last_funding_date               | date                        |           |          |                                          | plain    |              |
 closed_funding_on               | date                        |           |          |                                          | plain    |              |
 application_datetime            | timestamp without time zone |           |          |                                          | plain    |              |
 resolved_at                     | timestamp without time zone |           |          |                                          | plain    |              |
 created_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 updated_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 selected_partner_api_offer_id   | bigint                      |           |          |                                          | plain    |              |
 direct_mail_campaign_id         | bigint                      |           |          |                                          | plain    |              |
 selected_lead_id                | bigint                      |           |          |                                          | plain    |              |
 promise_cohort_id               | bigint                      |           |          |                                          | plain    |              |
 selected_pricing_output_id      | bigint                      |           |          |                                          | plain    |              |
 sfdc_funding_agreement_id       | character varying           |           |          |                                          | extended |              |
 pricing_id                      | bigint                      |           |          |                                          | plain    |              |
 initial_pre_application_id      | bigint                      |           |          |                                          | plain    |              |
 upstart_id                      | bigint                      |           |          |                                          | plain    |              |
 application_uuid                | uuid                        |           | not null |                                          | plain    |              |
 data_ownership_id               | bigint                      |           |          |                                          | plain    |              |
 upstart_application_id          | bigint                      |           |          |                                          | plain    |              |
 ownership_transferred_at        | timestamp without time zone |           |          |                                          | plain    |              |
 fingerprint_id                  | bigint                      |           |          |                                          | plain    |              |
 lang                            | character varying           |           |          | 'en'::character varying                  | extended |              |
 user_id                         | bigint                      |           |          |                                          | plain    |              |
Indexes:
    "applications_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "fk_rails_af99b547e9" FOREIGN KEY (pricing_id) REFERENCES pricings(id)
Child tables: pre_applications,
              user_applications
Access method: heap
```

UserApplications:

```SQL
                                                                    Table "public.user_applications"
             Column              |            Type             | Collation | Nullable |                 Default                  | Storage  | Stats target | Description
---------------------------------+-----------------------------+-----------+----------+------------------------------------------+----------+--------------+-------------
 id                              | bigint                      |           | not null | nextval('applications_id_seq'::regclass) | plain    |              |
 user_account_id                 | bigint                      |           |          |                                          | plain    |              |
 current_step                    | character varying           |           |          |                                          | extended |              |
 fee                             | integer                     |           |          |                                          | plain    |              |
 min_raising                     | integer                     |           |          |                                          | plain    |              |
 raising_amount                  | numeric(19,4)               |           |          |                                          | main     |              |
 max_loan_amount_at_signup       | integer                     |           | not null |                                          | plain    |              |
 initial_requested               | numeric(19,4)               |           |          |                                          | main     |              |
 initial_doc_number              | integer                     |           |          |                                          | plain    |              |
 contract_type                   | character varying           |           |          |                                          | extended |              |
 loan_funding_method             | character varying           |           |          |                                          | extended |              |
 turndown_partner                | character varying           |           |          |                                          | extended |              |
 use_of_funds                    | character varying           |           |          |                                          | extended |              |
 in_school                       | boolean                     |           |          | false                                    | plain    |              |
 is_psl                          | boolean                     |           |          | false                                    | plain    |              |
 compare_opt_in                  | boolean                     |           |          |                                          | plain    |              |
 last_decision_period_start_date | date                        |           |          |                                          | plain    |              |
 last_funding_date               | date                        |           |          |                                          | plain    |              |
 closed_funding_on               | date                        |           |          |                                          | plain    |              |
 application_datetime            | timestamp without time zone |           |          |                                          | plain    |              |
 resolved_at                     | timestamp without time zone |           |          |                                          | plain    |              |
 created_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 updated_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 selected_partner_api_offer_id   | bigint                      |           |          |                                          | plain    |              |
 direct_mail_campaign_id         | bigint                      |           |          |                                          | plain    |              |
 selected_lead_id                | bigint                      |           |          |                                          | plain    |              |
 promise_cohort_id               | bigint                      |           |          |                                          | plain    |              |
 selected_pricing_output_id      | bigint                      |           |          |                                          | plain    |              |
 sfdc_funding_agreement_id       | character varying           |           |          |                                          | extended |              |
 pricing_id                      | bigint                      |           |          |                                          | plain    |              |
 initial_pre_application_id      | bigint                      |           |          |                                          | plain    |              |
 upstart_id                      | bigint                      |           |          |                                          | plain    |              |
 application_uuid                | uuid                        |           | not null |                                          | plain    |              |
 data_ownership_id               | bigint                      |           |          |                                          | plain    |              |
 upstart_application_id          | bigint                      |           |          |                                          | plain    |              |
 ownership_transferred_at        | timestamp without time zone |           |          |                                          | plain    |              |
 fingerprint_id                  | bigint                      |           |          |                                          | plain    |              |
 lang                            | character varying           |           |          | 'en'::character varying                  | extended |              |
 user_id                         | bigint                      |           |          |                                          | plain    |              |
 pre_application_id              | bigint                      |           | not null |                                          | plain    |              |
 product_id                      | bigint                      |           |          |                                          | plain    |              |
 equifax_credit_report_id        | bigint                      |           |          |                                          | plain    |              |
 transunion_credit_report_id     | bigint                      |           |          |                                          | plain    |              |
 manual_status_change_at         | timestamp without time zone |           |          |                                          | plain    |              |
 decision_status                 | character varying           |           | not null | 'in_progress'::character varying         | extended |              |
 selected_loan_contract_id       | bigint                      |           |          |                                          | plain    |              |
 type                            | character varying           |           |          |                                          | extended |              |
Indexes:
    "index_user_applications_on_application_uuid" UNIQUE, btree (application_uuid)
    "index_user_applications_on_id" UNIQUE, btree (id)
    "index_user_applications_on_upstart_application_id" UNIQUE, btree (upstart_application_id)
    "index_user_applications_upstart_id_unique" UNIQUE, btree (upstart_id)
    "index_user_application_on_user_account_id" btree (user_account_id)
    "index_user_applications_on_contract_type" btree (contract_type)
    "index_user_applications_on_created_at" btree (created_at)
    "index_user_applications_on_data_ownership_id" btree (data_ownership_id)
    "index_user_applications_on_decision_status" btree (decision_status)
    "index_user_applications_on_direct_mail_campaign_id" btree (direct_mail_campaign_id)
    "index_user_applications_on_fingerprint_id" btree (fingerprint_id)
    "index_user_applications_on_initial_pre_application_id" btree (initial_pre_application_id)
    "index_user_applications_on_last_decision_period_start_date" btree (last_decision_period_start_date)
    "index_user_applications_on_loan_funding_method" btree (loan_funding_method)
    "index_user_applications_on_pre_application_id" btree (pre_application_id)
    "index_user_applications_on_product_id" btree (product_id)
    "index_user_applications_on_selected_lead_id" btree (selected_lead_id)
    "index_user_applications_on_selected_loan_contract_id" btree (selected_loan_contract_id)
    "index_user_applications_on_selected_partner_api_offer_id" btree (selected_partner_api_offer_id)
    "index_user_applications_on_selected_pricing_output_id" btree (selected_pricing_output_id)
    "index_user_applications_on_sfdc_funding_agreement_id" btree (sfdc_funding_agreement_id)
    "index_user_applications_on_user_id" btree (user_id)
Check constraints:
    "user_applications_lang_not_null" CHECK (lang IS NOT NULL) NOT VALID
Foreign-key constraints:
    "fk_rails_5a7bbb1e43" FOREIGN KEY (upstart_application_id) REFERENCES upstart_applications(id)
Referenced by:
    TABLE "deposit_account_creations" CONSTRAINT "fk_rails_0049d6b31d" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_publish_statuses" CONSTRAINT "fk_rails_00b73116c7" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_instant_plus_approval_statuses" CONSTRAINT "fk_rails_03042f7b3b" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "verification_calls" CONSTRAINT "fk_rails_077364cab9" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "auto_payoff_statuses" CONSTRAINT "fk_rails_0c0ef001ca" FOREIGN KEY (user_application_id) REFERENCES user_applications(id) NOT VALID
    TABLE "isa_statuses" CONSTRAINT "fk_rails_0cdc669391" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "income_shares" CONSTRAINT "fk_rails_0f1c939be0" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "underwriting_variables_for_loans" CONSTRAINT "fk_rails_1d087af155" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "final_approval_disclosures" CONSTRAINT "fk_rails_20c1922f4b" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "final_acceptances" CONSTRAINT "fk_rails_20cb544073" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "reorigination_events" CONSTRAINT "fk_rails_294b7ec80d" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "incomes" CONSTRAINT "fk_rails_2acc31966a" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "instant_plus_approvals" CONSTRAINT "fk_rails_2d6f85a4ec" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "fraud_reports" CONSTRAINT "fk_rails_4634a80d2c" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "ambassador_referrals" CONSTRAINT "fk_rails_598b05cbed" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "loan_contracts" CONSTRAINT "fk_rails_5a4be2e992" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "shortfall_payments" CONSTRAINT "fk_rails_5f08d1dc4c" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_instant_approval_statuses" CONSTRAINT "fk_rails_67af4323c9" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "one_time_origination_fee_refunds" CONSTRAINT "fk_rails_6a19c14d9c" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "deferral_periods" CONSTRAINT "fk_rails_6b28855f1a" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "metro_two_records" CONSTRAINT "fk_rails_6c8781ea61" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "bank_mobile_bank_account_opt_outs" CONSTRAINT "fk_rails_704ac50648" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_additional_loan_pricing_statuses" CONSTRAINT "fk_rails_756c6560cd" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "promotion_opts" CONSTRAINT "fk_rails_776331d976" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "additional_loan_eligibilities" CONSTRAINT "fk_rails_7c90f5d940" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "co_owner_consents" CONSTRAINT "fk_rails_7f73e97934" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_experiment_hard_pull_statuses" CONSTRAINT "fk_rails_7f7dcfd862" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "direct_credit_card_payoff_opt_statuses" CONSTRAINT "fk_rails_81a11eee46" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "deposit_account_opening_requests" CONSTRAINT "fk_rails_832b7ba657" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "offers" CONSTRAINT "fk_rails_89a42c5bde" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "job_offers" CONSTRAINT "fk_rails_98d0b32e70" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "funding_fail_events" CONSTRAINT "fk_rails_a10fa33bd1" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_hard_pull_statuses" CONSTRAINT "fk_rails_a87bc9b243" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "message_box_messages" CONSTRAINT "fk_rails_adda1c0f38" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "hardship_assistance_request_adverse_action_notices" CONSTRAINT "fk_rails_ae48e52428" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "cip_info_passports" CONSTRAINT "fk_rails_b125d78331" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "manual_verification_call_requests" CONSTRAINT "fk_rails_bb10a29408" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "bank_signup_discount_initiations" CONSTRAINT "fk_rails_be4a074943" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "initial_payment_preferences" CONSTRAINT "fk_rails_cffde558f9" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "upstart_tax_reconciliation_reports" CONSTRAINT "fk_rails_d053003a3b" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_tracking_pricing_statuses" CONSTRAINT "fk_rails_d07fb2eea8" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "cip_info_state_identifications" CONSTRAINT "fk_rails_d0a7e4f8dc" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "manual_test_score_verifications" CONSTRAINT "fk_rails_d3ba8b1064" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "estimated_approval_disclosures" CONSTRAINT "fk_rails_d76dd4246d" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "approvals" CONSTRAINT "fk_rails_d7e7b1b5d9" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "alternate_metro_two_records" CONSTRAINT "fk_rails_e2f0a1fc4a" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "instant_approvals" CONSTRAINT "fk_rails_e39e6048d9" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "user_provided_permissions" CONSTRAINT "fk_rails_e7acd8ff52" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "requirement_deadlines" CONSTRAINT "fk_rails_eaf966bbae" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "disclosures" CONSTRAINT "fk_rails_ecf58ea225" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "lien_transfer_applications" CONSTRAINT "fk_rails_f056f675ff" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "instant_approval_small_dollars" CONSTRAINT "fk_rails_f108ce4e51" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "marqeta_users" CONSTRAINT "fk_rails_f2106ae195" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "plaid_id_verification_results" CONSTRAINT "fk_rails_f232618779" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "background_requirements_generation_statuses" CONSTRAINT "fk_rails_fc41e2efe8" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
    TABLE "inquiry_disputes" CONSTRAINT "fk_rails_fed5417999" FOREIGN KEY (user_application_id) REFERENCES user_applications(id)
Inherits: applications
Access method: heap
```

PreApplications:

```SQL
                                                                     Table "public.pre_applications"
             Column              |            Type             | Collation | Nullable |                 Default                  | Storage  | Stats target | Description
---------------------------------+-----------------------------+-----------+----------+------------------------------------------+----------+--------------+-------------
 id                              | bigint                      |           | not null | nextval('applications_id_seq'::regclass) | plain    |              |
 user_account_id                 | bigint                      |           |          |                                          | plain    |              |
 current_step                    | character varying           |           |          |                                          | extended |              |
 fee                             | integer                     |           |          |                                          | plain    |              |
 min_raising                     | integer                     |           |          |                                          | plain    |              |
 raising_amount                  | numeric(19,4)               |           |          |                                          | main     |              |
 max_loan_amount_at_signup       | integer                     |           | not null |                                          | plain    |              |
 initial_requested               | numeric(19,4)               |           |          |                                          | main     |              |
 initial_doc_number              | integer                     |           |          |                                          | plain    |              |
 contract_type                   | character varying           |           |          |                                          | extended |              |
 loan_funding_method             | character varying           |           |          |                                          | extended |              |
 turndown_partner                | character varying           |           |          |                                          | extended |              |
 use_of_funds                    | character varying           |           |          |                                          | extended |              |
 in_school                       | boolean                     |           |          | false                                    | plain    |              |
 is_psl                          | boolean                     |           |          | false                                    | plain    |              |
 compare_opt_in                  | boolean                     |           |          |                                          | plain    |              |
 last_decision_period_start_date | date                        |           |          |                                          | plain    |              |
 last_funding_date               | date                        |           |          |                                          | plain    |              |
 closed_funding_on               | date                        |           |          |                                          | plain    |              |
 application_datetime            | timestamp without time zone |           |          |                                          | plain    |              |
 resolved_at                     | timestamp without time zone |           |          |                                          | plain    |              |
 created_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 updated_at                      | timestamp without time zone |           | not null |                                          | plain    |              |
 selected_partner_api_offer_id   | bigint                      |           |          |                                          | plain    |              |
 direct_mail_campaign_id         | bigint                      |           |          |                                          | plain    |              |
 selected_lead_id                | bigint                      |           |          |                                          | plain    |              |
 promise_cohort_id               | bigint                      |           |          |                                          | plain    |              |
 selected_pricing_output_id      | bigint                      |           |          |                                          | plain    |              |
 sfdc_funding_agreement_id       | character varying           |           |          |                                          | extended |              |
 pricing_id                      | bigint                      |           |          |                                          | plain    |              |
 initial_pre_application_id      | bigint                      |           |          |                                          | plain    |              |
 upstart_id                      | bigint                      |           |          |                                          | plain    |              |
 application_uuid                | uuid                        |           | not null |                                          | plain    |              |
 data_ownership_id               | bigint                      |           |          |                                          | plain    |              |
 upstart_application_id          | bigint                      |           |          |                                          | plain    |              |
 ownership_transferred_at        | timestamp without time zone |           |          |                                          | plain    |              |
 fingerprint_id                  | bigint                      |           |          |                                          | plain    |              |
 lang                            | character varying           |           |          | 'en'::character varying                  | extended |              |
 user_id                         | bigint                      |           |          |                                          | plain    |              |
 pbu_program_id                  | bigint                      |           |          |                                          | plain    |              |
 product_type_id                 | bigint                      |           | not null |                                          | plain    |              |
Indexes:
    "index_pre_applications_on_id" UNIQUE, btree (id)
    "index_pre_applications_on_user_id_unique_by_sdl_product_type" UNIQUE, btree (user_id) WHERE product_type_id = 1
    "index_pre_application_on_user_account_id" btree (user_account_id)
    "index_pre_applications_on_application_uuid" btree (application_uuid)
    "index_pre_applications_on_contract_type" btree (contract_type)
    "index_pre_applications_on_created_at" btree (created_at)
    "index_pre_applications_on_data_ownership_id" btree (data_ownership_id)
    "index_pre_applications_on_direct_mail_campaign_id" btree (direct_mail_campaign_id)
    "index_pre_applications_on_fingerprint_id" btree (fingerprint_id)
    "index_pre_applications_on_initial_pre_application_id" btree (initial_pre_application_id)
    "index_pre_applications_on_last_decision_period_start_date" btree (last_decision_period_start_date)
    "index_pre_applications_on_loan_funding_method" btree (loan_funding_method)
    "index_pre_applications_on_pbu_program_id" btree (pbu_program_id)
    "index_pre_applications_on_selected_lead_id" btree (selected_lead_id)
    "index_pre_applications_on_selected_partner_api_offer_id" btree (selected_partner_api_offer_id)
    "index_pre_applications_on_selected_pricing_output_id" btree (selected_pricing_output_id)
    "index_pre_applications_on_sfdc_funding_agreement_id" btree (sfdc_funding_agreement_id)
    "index_pre_applications_on_upstart_application_id" btree (upstart_application_id)
    "index_pre_applications_on_user_id" btree (user_id)
    "index_pre_applications_upstart_id" btree (upstart_id)
Check constraints:
    "pre_applications_lang_not_null" CHECK (lang IS NOT NULL) NOT VALID
Foreign-key constraints:
    "fk_rails_84c968ca6b" FOREIGN KEY (product_type_id) REFERENCES product_types(id)
    "fk_rails_d1aa475ffa" FOREIGN KEY (pbu_program_id) REFERENCES pbu_programs(id)
Referenced by:
    TABLE "additional_loan_eligibilities" CONSTRAINT "fk_rails_457dba213b" FOREIGN KEY (pre_application_id) REFERENCES pre_applications(id)
    TABLE "auto_refi_imputed_data_apps" CONSTRAINT "fk_rails_7a2e0f7c06" FOREIGN KEY (pre_application_id) REFERENCES pre_applications(id)
    TABLE "leads" CONSTRAINT "fk_rails_e85df46fe8" FOREIGN KEY (pre_application_id) REFERENCES pre_applications(id) NOT VALID
Inherits: applications
Access method: heap
```
