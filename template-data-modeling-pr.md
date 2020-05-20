# Overview
_Summarize the goal of your work and what motivated it_

# Key changes
- _Describe major updates_

# Other notes
- _List open questions and other things your reviewer should know_

# Related links
- _List links to related ticket(s)_
- _(optional) Link to other files and resources, such as specific related reports in a BI tool_

# Checklist
- [ ] All models, whether directly changed by this PR or not, run successfully
- [ ] New code follows [dbt coding conventions](https://github.com/brooklyn-data/co/blob/master/dbt_coding_conventions.md) and [SQL style guide](https://github.com/brooklyn-data/co/blob/master/sql_style_guide.md)
- [ ] Testing
    - [ ] All added/modified models and columns are documented and tested in `schema.yml` files
    - [ ] All tests pass **OR**
    _Check one:_
    - [ ] There are no new test failures, whether or not the existing model was intentionally changed for this PR
    - [ ] This PR causes new test failures and there is a plan in place for addressing them
