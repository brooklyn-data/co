## Overview
_Summarize the goal of your work and what motivated it._

## Key changes
- _Describe major updates. If multiple models or layers of models are added or updated, break up those layers/models into subheaders within this section._
- _If applicable, include a screenshot of the affected parts of the lineage graph._

## Validation
- _Describe the validation process underwent to validate the changes_
- _If applicable, include any validation queries or dbt execution commands completed_

<details>
  <summary>SQL validation queries</summary>

  Paste your SQL queries here
  ```
  select * from table
  ```

</details>

## Areas for reviewers to focus on
- _Highlight any outstanding questions related to your PR or any parts you'd like reviewers to take a closer look at_

## Future considerations
- _Call out any TODOs, known limitations, technical debt, or improvements you'd like to make at a later date_

## Deployment Notes
- _Specify any deployment notes or downstream impacts reviewers should be aware of_

## Other notes
- _List any other notes that don't fit into any of the above sections_

## Related links
- _List links to related ticket(s)._
- _(optional) Link to other files and resources, such as specific related reports in a BI tool or documentation._

## Checklist
- [ ] All models, whether directly changed by this PR or not, run successfully.
- [ ] New code follows [dbt coding conventions](https://github.com/brooklyn-data/co/blob/master/dbt_coding_conventions.md) and [SQL style guide](https://github.com/brooklyn-data/co/blob/master/sql_style_guide.md).
- [ ] Testing:
    - [ ] All added/modified models and columns are documented and tested in `schema.yml` files.
    - [ ] All tests pass. **OR**
    _Check one:_
    - [ ] There are no new test failures, whether or not the existing model was intentionally changed for this PR.
    - [ ] This PR causes new test failures and there is a plan in place for addressing them.
