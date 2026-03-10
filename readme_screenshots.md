📹 Demo video: https://us06web.zoom.us/clips/share/b2ucu4zJRCGT2EU9DtriNA

# Casa Agent – Walkthrough Screenshots

This page shows the workflow of the **Casa Agent multi-agent system**, which evaluates property listings against a user preference profile.

---

## 1. Project Structure

Overview of the repository structure.

![Repository structure](screenshots/01_casa-agent-repository-structure.png)

---

## 2. Agent Instructions

Example of the instructions given to the agents that evaluate listings.

![Agent instructions](screenshots/02_casa-agent-instructions.png)

---

## 3. User Preference Profile – House Details

User defines preferences about the type of property they are looking for.

![House details](screenshots/03_casa-agent_profile-house-details.png)

---

## 4. User Preference Profile – Exposure

Preferred sunlight exposure and orientation.

![Exposure preferences](screenshots/04_casa-agent_profile-exposure.png)

---

## 5. User Preference Profile – Commute

User specifies important commute destinations.

![Commute preferences](screenshots/05_casa-agent_profile-commute.png)

---

## 6. Save User Profile

Saving the preference profile used by the evaluation agents.

![Save profile](screenshots/06_casa-agent_profile-save.png)

---

## 7. Evaluate Listing

A property listing is submitted for analysis by the agent system.

![Evaluate listing](screenshots/07_casa-agent_evaluate-listing.png)

---

## 8. Example Result – Apartment 1 (Recommended)

The system evaluates the listing and determines it is worth visiting.

![Apartment recommended](screenshots/08_results-apartment1-yes.png)

---

## 9. Financial Evaluation

The financial agent evaluates affordability and price metrics.

![Financial evaluation](screenshots/09_results-apartment1-yes-finances.png)

---

## 10. Sunlight / Exposure Evaluation

The exposure agent evaluates sunlight and orientation.

![Sunlight evaluation](screenshots/10_results-apartment1-yes-sunlight.png)

---

## 11. Commute Evaluation

The commute agent evaluates travel time to important destinations.

![Commute evaluation](screenshots/11_results-apartment1-yes-commutability.png)

---

## 12. Example Result – Apartment 2 (Not Recommended)

The system determines the listing does not meet the user's criteria.

![Apartment rejected](screenshots/12_results-apartment2-no-go.png)

---

## 13. Evaluation History

The system keeps a history of previous evaluations.

![History](screenshots/13_history.png)
