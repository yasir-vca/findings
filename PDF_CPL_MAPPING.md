# VoyagerGo PDF/Document CPL Mapping

Date: 2026-05-04
Owner: TODO
Scope: Frontend call sites that trigger PDF/document/template generation or retrieval relevant to localization.

## Status Legend

- already passes CPL: explicit language/CPL is passed in query/body/form field
- needs change: no explicit language/CPL is passed (relies on implicit/default behavior)
- needs deeper trace: frontend call found, but backend/indirect path must be validated before deciding if change is required
- out of scope: related document/image upload or preview path exists, but it does not generate a PDF/document template in the RN frontend

## Global Context

- `Accept-Language` header is set centrally in `src/networking/request.ts` via `config.headers[MVHHeaders.ACCEPT_LANGUAGE] = getLocale() || ''`.
- This does not guarantee backend document rendering uses language/CPL unless endpoint contract explicitly honors header-based localization.
- Several story filenames from the ticket do not exist verbatim in this RN repo. Where that happened, the closest current frontend equivalent is called out explicitly below.

## Consolidated Mapping

| Domain                          | UI/Trigger                                          | File                                                                                                                      | Method                                            | Endpoint                                                                                  | CPL Status         | Evidence                                                                                  |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------- |
| Finance (Estimates)             | Estimate details action button (Finalized -> print) | `src/screens/PetPreviewScreen/OrdersAndEstimates/Estimates/EstimateDetails/EstimateActionButton/EstimateActionButton.tsx` | `handleOnPress` -> `fetchPdf(...)`                | `orderestimateorchestration/v1/api/print/{orderId}?accountId=...&patientId=...&petId=...` | needs change       | Calls `useEstimatePrintPdf` without language param in payload.                            |
| Finance (Estimates API)         | Hook/API layer                                      | `src/networking/orderEstimateOrchestration/orderEstimateOrchestrationApis.ts`                                             | `printPdf`                                        | `mvhApiPaths.estimatePrintPdf(...)`                                                       | needs change       | Method forwards only `orderId/accountId/patientId/petId`.                                 |
| Finance ECM Doc                 | Estimate details document retrieval                 | `src/networking/ecm/ecmApi.ts`                                                                                            | `getEcmDocument`                                  | `ecm/v1/api/documents/{documentId}`                                                       | needs deeper trace | No explicit language field; must confirm endpoint behavior with Accept-Language.          |
| Wellness Contract               | Sign contract flow (open unsigned contract)         | `src/components/PetPreviewScreen/WellnessScreen/WellnessCurrentPlanInfo/WellnessCurrentPlanInfo.tsx`                      | `getContractToSignOnUserAction`                   | `wellnessplanlifecycleorchestration/v1/api/enrollments/{enrollmentId}/contract`           | needs change       | Calls `getContract({ enrollmentId })` only.                                               |
| Wellness Contract (API)         | Contract retrieval                                  | `src/networking/wellnessContractSignature/wellnessContractSignatureApis.ts`                                               | `getContract`                                     | `mvhApiPaths.wellnessGetContract(enrollmentId)`                                           | needs change       | No query/body language param.                                                             |
| Wellness Contract Sign          | Sign/submit signed agreement                        | `src/components/PetPreviewScreen/WellnessScreen/WellnessCurrentPlanInfo/WellnessCurrentPlanInfo.tsx`                      | `onDocumentViewerSave` -> `postSignContract(...)` | `.../signcontract-v3`                                                                     | needs change       | FormData includes agreement/appointment/visit/associate fields only.                      |
| Wellness Contract Sign (API)    | FormData builder                                    | `src/networking/wellnessContractSignature/wellnessContractSignatureApis.ts`                                               | `signContract`                                    | `mvhApiPaths.wellnessSignContract(enrollmentId)`                                          | needs change       | `getSignContractFormData` has no language field.                                          |
| Wellness Contract Preview       | View already-signed contract                        | `src/components/PetPreviewScreen/WellnessScreen/WellnessCurrentPlanInfo/WellnessCurrentPlanInfo.tsx`                      | `getSignedContractOnUserAction`                   | `documentmanagement/v1/api/documents?filePath=...`                                        | needs change       | No explicit CPL/language param in query.                                                  |
| Wellness Contract Preview (API) | Fetch document bytes                                | `src/networking/wellnessContractPreview/wellnessContractPreviewApi.ts`                                                    | `getWellnessContractAttachmentPreview`            | `mvhApiPaths.documentsDataUrl?filePath=...`                                               | needs change       | Query contains only `filePath`.                                                           |
| Visit Digital Forms (PVQ/ATPC)  | Checklist online form task opens external form URL  | `src/hooks/useCheckList.ts`                                                                                               | `handleOnlineForm` -> `fetchDocumentUrl(...)`     | `digitalformsorchestration/v1/api/forms/appointment/geturl`                               | needs change       | Request body includes account/appointment/patient/petDetails/BU/OU fields, but no CPL.    |
| Signature/Digital Forms         | Generate form URL/content for signature             | `src/networking/digitalFormOrchestration/generateForm/generateFormApis.ts`                                                | `generateForm`                                    | `digitalformsorchestration/v1/api/url/generate`                                           | already passes CPL | Request body includes `acceptLanguage: getLocale()`.                                      |
| Signature/Digital Forms         | Fetch generated signature form                      | `src/networking/digitalForms/getSignatureForms/getSignatureFormsApis.ts`                                                  | `getSignatureForm`                                | `digitalforms/v1/api/forms/getsignatureforms?uniqueFormId=...`                            | needs deeper trace | No explicit language; may rely on language captured in generate step.                     |
| Signature/Digital Forms         | Submit signed form                                  | `src/networking/digitalForms/submitSignatureForms/submitSignatureFormsApis.ts`                                            | `submitSignatureForm`                             | `digitalforms/v1/api/forms/submit/digitalforms-contract?uniqueFormId=...`                 | needs deeper trace | No language field in FormData; confirm whether language is bound to uniqueFormId session. |
| Signature/Doc Queue             | Signature inbox list                                | `src/networking/documentQueueManagement/search/searchApis.ts`                                                             | `getDocumentQueue`                                | `documentqueuemanagement/v1/api/docqueue/search`                                          | needs change       | Payload does not include language/CPL.                                                    |
| Medical Record Docs             | Create patient document metadata                    | `src/networking/patientDocuments/patientDocumentsApi.ts`                                                                  | `createPatientDocument`                           | `medicalrecordmanagement/v1/api/patientdocuments`                                         | needs change       | Hardcoded `language: 'en'` in `displayHealthConcerns`.                                    |
| Medical Record Docs             | Upload patient document metadata                    | `src/networking/patientDocumentData.ts`                                                                                   | `getPatientDocument`                              | `medicalrecordmanagement/v1/api/patientdocuments`                                         | needs change       | Hardcoded `language: 'en'` in `displayHealthConcerns`.                                    |

## Story-Specific Coverage and Gaps

### Scenario 2: Finance

- Covered:
  - Estimate print trigger and API path identified.
  - ECM document retrieval path identified.
- Gaps to close:
  - Determine if finance print endpoint accepts explicit `cpl` or `language` query parameter.

### Scenario 3: Non-Finance Requested Domains

- WellnessPlans: covered (contract/get/sign/preview).
- ClientAndPatient `client-connections.service.ts`: exact file/path not found in current workspace. Closest related RN paths are communications attachment upload helpers in `src/networking/communicationService/communicationServiceApis.ts`, but they upload stored attachments and do not generate document templates.
- Visit PVQ/ATPC triggers: covered by checklist online-form flow in `src/hooks/useCheckList.ts` calling `digitalformsorchestration/v1/api/forms/appointment/geturl`. Supporting helper endpoints also exist in `src/networking/digitalFormOrchestration/digitalFormOrchestrationApi.ts`:
  - `appointment/v1/api/appointments/pvqparams?appointmentId=...` via `getDigitalFormAppointmentParams`
  - `digitalformsorchestration/v1/api/form/status?petAppointmentId=...` via `getDigitalFormStatusFromAppointment`
    These two helpers are status/metadata calls, not the actual document/template generation trigger.
- ScheduleAndAppointment reminders: no reminder PDF/document-template generation call site found in the RN frontend under `src`.
- MedicalRecord AVS `HttpService.BuildHeaders`: no `BuildHeaders` symbol found in current TS code under `src`; current RN equivalent is centralized request header construction in `src/networking/request.ts`.
- Pharmacy Rx print triggers: no explicit frontend Rx print/PDF generation call found. Pharmacy usage in this repo is limited to prescription retrieval and product/code search (`src/networking/prescriptions/prescriptionsApi.ts`, `src/networking/pharmacyManagement/searchProducts/searchProductsApis.ts`, `src/networking/pharmacyOrchestration/searchCodes/searchCodesApis.ts`).

### Scenario 4: Needs Deeper Trace

The following are currently tagged needs deeper trace:

- `src/networking/ecm/ecmApi.ts` -> `getEcmDocument`
- `src/networking/digitalForms/getSignatureForms/getSignatureFormsApis.ts` -> `getSignatureForm`
- `src/networking/digitalForms/submitSignatureForms/submitSignatureFormsApis.ts` -> `submitSignatureForm`

Downstream trace targets:

- Backend contracts for the above endpoints to confirm whether Accept-Language header is authoritative.
- Whether `uniqueFormId` captures language in orchestration and propagates across get/submit stages.

## Related Paths Reviewed But Excluded From Generator Scope

| Domain / Area                   | File                                                                  | Why excluded                                                                                                                        |
| ------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Patient attachment preview      | `src/screens/PatientAttachmentScreen.tsx`                             | Opens an already-stored attachment via `documentmanagement/v1/api/documents?contentType=true&filePath=...`; no template generation. |
| Communication attachment view   | `src/components/Communications/MessageList/MessageItems/TextItem.tsx` | Downloads attachment previews for message media thumbnails; no template generation.                                                 |
| Patient avatar image fetch      | `src/components/PatientCard/PatientCardAvatar.tsx`                    | Uses `src/networking/documentsData.ts` for profile images only.                                                                     |
| Patient document uploads        | `src/networking/documentManagement/documentManagementApi.ts`          | Upload/confirm storage operations only; not PDF/template generation.                                                                |
| Communication attachment upload | `src/networking/communicationService/communicationServiceApis.ts`     | Upload/confirm storage operations only; not PDF/template generation.                                                                |

## Ticket-Ready Summary

- In-scope frontend inventory identified: 16 call paths.
- Status breakdown: 1 already passes CPL, 12 need change, 3 need deeper trace.
- Exact story-name equivalents not present in this RN repo: `client-connections.service.ts`, reminders PDF generation, AVS `HttpService.BuildHeaders`, and Pharmacy Rx print.
- Closest current frontend equivalents were mapped where they exist, especially visit PVQ/ATPC online forms through checklist -> `digitalformsorchestration` appointment `geturl`.
- Clearest implementation defect found in current frontend code: patient document metadata hardcodes `language: 'en'` in both `src/networking/patientDocuments/patientDocumentsApi.ts` and `src/networking/patientDocumentData.ts`.

## Follow-Up If Engineering Change Is Requested

- Replace hardcoded `language: 'en'` with locale-derived CPL in patient document metadata builders.
- Add explicit CPL/language to finance estimate print, wellness contract get/sign/preview, visit appointment digital form `geturl`, and document queue request payloads if backend contracts support it.
- Confirm whether ECM and signature get/submit endpoints already inherit language through header/session semantics before changing request shapes.
