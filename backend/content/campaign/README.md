# Campaign (Content API)

- Campaign is a collection of exams and questions; a list of exams and conditions to advance to the next exam. (See [Campaign](../../../general/components/Campaign.md) for more details.)
- Used to create and manage multi-exam flows with progress and conditions.

## Endpoints

- `GET /Campaign/` - Get campaigns by parameters
- `POST /Campaign/` - Create or update campaign
- `DELETE /Campaign/` - Delete campaign
- `POST /Campaign/livetest` - Enable live testing and result collection for the campaign and produce a link to start the campaign for the user
