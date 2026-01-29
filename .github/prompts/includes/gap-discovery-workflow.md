# Gap Discovery Workflow (Review Prompts)

This standardized workflow applies to all review prompts that may modify the artifact being reviewed.

## Process Steps

1. **List ALL gaps discovered** with specific section/line references
2. **For each gap, provide a suggested fix** (concrete text, design change, or specification)
3. **Present full gap list with fixes to user**
4. **Ask user:** "Apply these fixes?" or "Would you like to adjust any suggestions?"
5. **Modify artifact ONLY after explicit user approval**

## Key Principles

- ✅ Review prompts MAY modify artifacts with user approval
- ❌ Never modify automatically without user consent
- ✅ All suggested fixes must be concrete and actionable
- ✅ Severity must be assigned: BLOCKING / HIGH / MEDIUM / LOW
- ✅ Gap locations must cite specific sections/lines/components
