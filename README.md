# (Re)Sorcerer

**Link:** Innovation Studio  

---

## Executive Summary  
**(Re)Sorcerer** is an AI-powered staffing agent designed to automate and optimise the matching of delivery resources to project opportunities.  
It reduces manual effort, accelerates staffing, and ensures higher quality matches across ISD and beyond.

---

## Business Case  

Currently, staffing projects involves numerous manual touchpoints in a lengthy end-to-end process (from Pursuit Leads through to final staffing).  
This creates inefficiency, delays, and mismatches.

**Key Benefits**:  
- ⏱ Reduced time-to-staff opportunities  
- 🎯 Improved matching quality  
- 📈 Scalable across Microsoft orgs and beyond  

**Long-Term Potential**:  
- Expand outside ISD into other Microsoft organisations  
- Operate externally as a Recruitment Fulfilment Agent  
- Assist recruiters and talent agencies by matching CVs with vacancies  

---

## Vision, Use-Cases & MVP Capabilities  

Using Microsoft ISD as a case study, (Re)Sorcerer aims to provide a **frictionless approach** for:  
- **Delivery Resources** → Discovering and selecting relevant opportunities.  
- **Recruitment Managers** → Identifying skilled Delivery Resource matches.  

### Challenges Addressed  
- Skilling profile accuracy and maintenance is currently cumbersome.  
- Project sales documents (SOWs, artefacts) lack structured extraction for staffing use.  

### MVP Approach  
- RMs upload SoWs.  
- (Re)Sorcerer extracts critical fields (roles, skills, hours, dates).  
- Data is stored in a register (SharePoint for MVP; Dataverse/Cosmos for scale).  
- Delivery Resources query opportunities or update their skills.  
- AI ensures matches are skills-appropriate.  

---

## Work Flow Architecture (MVP)  

**Resource Manager Flow**  
1. Upload SoW to (Re)Sorcerer.  
2. (Re)Sorcerer extracts and stores:  
   - Role discipline & count  
   - Skills required  
   - Hours, dates, duration  
3. AI matches resources based on skills/availability.  
4. Potential consultants are returned to RM.  

**Delivery Resource Flow**  
- Ask queries like:  
  - “What opportunities are available?”  
  - “Update my skills.”  
- (Re)Sorcerer queries the register and returns matched results.  

---

## Roadmap & Future Capabilities  

Future developments to increase enterprise value:  
- 🔗 Integration with **Compass One**  
- 🔗 Integration with **ESXP**  
- 🔔 Enhanced notifications & approval gates (Email/Teams)  

This will expand adoption across the end-to-end staffing process and reduce manual touchpoints.  

---

## Considerations  

Key topics identified during the Hackathon:  

| **Scenario** | **Approach** |
|--------------|--------------|
| Security Cleared projects | Use isolated databases; role-based security review required |
| Test Data & Environments | Bogus data used for Hackathon; need proper test datasets & sandboxes |
| Project Managers’ use | PMs should be able to query and match in the same way as consultants |
| Learning/Shadowing | Add fields to support junior staff development opportunities |

---

## Open Questions  

| **Question** | **Rationale** |
|--------------|---------------|
| Can Test Data be provided for future development? | Needed to scale beyond hackathon proof-of-concept |
| Can Test environments be provided for development? | Ensures safe and consistent testing |
| Can integration environments (Compass One, ESXP) be provided? | Allows real validation of system connectivity |
| End-to-End sales process documentation? | Prevents knowledge gaps & redevelopment blockers |
| How are Project Managers resourced? | Clarification of process needed |
| Stakeholder map? | Required for governance & adoption planning |

---

## Closed Questions  

During the Hackathon, several questions were raised and resolved. These are documented in internal notes for record-keeping.  

---

## Contributors  
Hackathon development by the **Innovation Studio** team.  

---
