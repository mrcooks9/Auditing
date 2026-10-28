# ESP Auditing Onboarding Scenario Table

## Proposed scenarios

| What | Where | Category | Component | Activity | Actor | Operation | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Create custom ESP policy | Graph API | Enrollment | Enrollment | Create Windows10EnrollmentCompletionPageConfiguration | ITPro / Application | Create | Creates a custom Enrollment Status Page policy. |
| Update custom ESP display name | Graph API | Enrollment | Enrollment | Patch Windows10EnrollmentCompletionPageConfiguration | ITPro / Application | Patch | Metadata change on a custom ESP policy. |
| Update custom ESP setting | Graph API | Enrollment | Enrollment | Patch Windows10EnrollmentCompletionPageConfiguration | ITPro / Application | Patch | Setting change on a custom ESP policy. |
| Update default ESP policy | Graph API | Enrollment | Enrollment | Patch Windows10EnrollmentCompletionPageConfiguration | ITPro / Application | Patch | Covers the customer scenario where the default ESP timeout or another setting was changed. |
| Delete custom ESP policy | Graph API | Enrollment | Enrollment | Delete Windows10EnrollmentCompletionPageConfiguration | ITPro / Application | Delete | Default ESP policies cannot be deleted. |
| Change custom ESP policy priority | Graph API | Enrollment | Enrollment | Action DeviceEnrollmentConfiguration | ITPro / Application | Action | Follows the existing SetPriority auditing pattern used by other enrollment configuration policies. |
| Replace enrollment policy assignments | Graph API | Enrollment | Enrollment | Assign DeviceEnrollmentConfiguration | ITPro / Application | Action | Implemented by the common enrollment configuration assignment API and applies to all supported enrollment policy types, including ESP. |

The actor should be derived from the Graph execution context. It will normally be an
ITPro for delegated administrator requests or an Application for app-only requests.

## Scope and ownership

### Included in the proposed onboarding plan

- Create, patch, and delete auditing for custom ESP policies.
- Patch auditing for the default ESP policy.
- SetPriority auditing for custom ESP policies, for parity with enrollment limit,
  platform restriction, Windows Hello for Business, and single-platform restriction
  policies.
- Scope-tag changes made through PATCH; these do not require a separate activity.
- Assignment auditing in the common enrollment configuration assignment API.

### Assignment ownership and scope

Assignment replacement is implemented in the common
`DeviceEnrollmentConfigurationRoutingBehavior.UpdateConfigAssignmentsAsync` path.
That path is used across enrollment configuration policy types and currently has no
`GraphApiAuditor` integration. It is included in this plan so the gap can be
addressed, but the API is technically owned by the Enrollment team and the missing
auditing is common to all enrollment configuration policies rather than specific to
ESP. The audit event should record the added and removed assignment targets,
including assignment filters.

## Implementation alignment

- Use `GraphApiAuditor<Windows10EnrollmentCompletionPageConfiguration>` with
  component `"Enrollment"`, `AuditCategory.Enrollment`, and
  `AuditingTag.MonoOnboardingService`.
- Follow the existing OnboardingService behavior-layer pattern using
  `AuditingHelper.Audit`; do not introduce the illustrative `IAuditLogger`,
  `[Auditable]`, or repository-based controller design from the proposal.
- Emit audit records only after the corresponding persistence operation succeeds.
- Use the Graph entity class `Windows10EnrollmentCompletionPageConfiguration`.
  `DEWECPConfiguration.cs` is its filename, not a type named
  `DEWECPConfiguration`.
- Confirm the final activity names and handling of `CustomErrorMessage` and large
  collections with the Auditing team.

## Proposal corrections

The proposal correctly identifies the auditing gap and the Enrollment category, but
the following details should be corrected before it is treated as an implementation
design.

### Required implementation corrections

1. **Use the existing FSI auditing pattern.** OnboardingService enrollment
   behaviors manually construct `GraphApiAuditor<T>` and invoke it through
   `AuditingHelper.Audit`. The proposal's `IAuditLogger`, `[Auditable]`,
   controller dependency injection, and repository examples are illustrative and
   do not match this service.
2. **Use the correct entity type.**
   `GraphApiAuditor<Windows10EnrollmentCompletionPageConfiguration>` should be
   used for ESP-specific activity names. `DEWECPConfiguration.cs` is a filename;
   there is no `DEWECPConfiguration` class.
3. **Use the existing Enrollment component.** The ESP events should use component
   string `"Enrollment"`, consistent with the other enrollment configuration
   policies. `AuditingComponentType.Enrollment` does not exist, and this work
   should not introduce a new `EnrollmentStatusPage` component.
4. **Do not treat MessageBus onboarding as a new ESP prerequisite.**
   OnboardingService already publishes enrollment audit records using
   `AuditingTag.MonoOnboardingService`. MessageBus registration should only be
   revisited if end-to-end validation shows a publishing problem.
5. **Add auditing in both mutation paths.** Custom ESP create, patch, delete, and
   SetPriority operations are implemented in `DEWCPBehavior.cs`. Default ESP
   patching is implemented separately in `DefaultDECBehavior.cs`.
6. **Audit only successful mutations.** Place `AuditCreate`, `AuditPatch`,
   `AuditDelete`, and `AuditEntityAction` calls after the corresponding storage
   operation succeeds.
7. **Capture the pre-update entity for PATCH.** Both custom and default ESP patch
   paths need the existing Graph entity before persistence so
   `GraphApiAuditor.AuditPatch` can populate old and new property information.
8. **Correct the property scope.** The proposal's list omits
   `AllowLogCollectionOnInstallFailure` and `InstallQualityUpdates`. Scope-tag
   changes are also supported through `RoleScopeTagIds`, but remain part of the
   normal Patch activity rather than a separate activity.
9. **Handle sensitive and collection properties deliberately.**
   `CustomErrorMessage` should be recorded as changed without logging its raw old
   or new text. `SelectedMobileAppIds` should retain the application IDs because
   they are necessary to understand the policy change and are bounded by the ESP
   limit of 100 applications.
10. **Implement assignment auditing in the common path.** Assignments are changed
    through `UpdateConfigAssignmentsAsync`. Include that work in this plan, while
    recording that the API is owned by the Enrollment team and the gap affects all
    enrollment configuration policy types rather than ESP alone.

### Items requiring Auditing-team confirmation

- The supported framework pattern for recording
  `CustomErrorMessageChanged = true` without publishing the raw text. The
  recommendation is to suppress the value; Auditing-team input is only needed to
  confirm the preferred API or property shape.

### Resolved property-handling recommendation

- `SelectedMobileAppIds` does not require a privacy exception. The values are
  application GUIDs rather than free-form customer text, and the policy enforces
  a maximum of 100 entries. Use the standard framework representation, consistent
  with the other audited enrollment configuration properties.
