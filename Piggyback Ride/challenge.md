**Company:** Forgeline Software, a vendor of rapid admin-panel tooling for
PHP teams.

**Project:** **PanelSmith**, a Composer package that scaffolds full-featured
admin panels on top of a popular PHP framework. It ships ready-made
controllers, views, and form requests, including a "My Account" page where
authenticated users edit their personal info. The page has two separate,
deliberate flows: an *account info* form (name, email) and a *change
password* form that requires re-entering the current password before the
new one is accepted.

**What the code does:**
`src/app/Http/Controllers/MyAccountController.php` serves both flows.
`postAccountInfoForm` receives an `AccountInfoRequest` and persists the
submitted data onto the logged-in user model; `postChangePasswordForm`
handles password changes with proper verification.

**Your task:** You are a security consultant auditing a customer's PanelSmith-
based back office. The customer's policy is strict: *no password may be set
without proving knowledge of the current one.* Your job is to review the
account-info save path and determine whether the two-form separation is
actually enforced at the code level — or whether a crafted request can slip
a new password through the "harmless" profile form.

**Scope note:** Focus entirely on
`src/app/Http/Controllers/MyAccountController.php`, in particular how
`postAccountInfoForm` builds the attributes array it hands to the user
model's `update()`. Compare it with what `postChangePasswordForm` does.
