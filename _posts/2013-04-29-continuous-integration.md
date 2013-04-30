---
layout: post
title: Continuous Integration Reminders
---

# Best Practice Reminders

## Continuous Integration and Software Development Infrastructure

This is a list of best practices for Continuous Integration and Software Development.  Derived from a list by Barry Jaspan.

1. Use a source code repository. This is step zero for good software development.

2. Make small, frequent changes. All developers should commit their changes frequently. This reduces the inevitable conflicts and lets problems surface sooner. Also, small, frequent changes enable small, frequent releases, making all the rest of the principles more valuable.

3. Automate testing. Have your repository automatically integrated with a testing environment, so that every commit triggers a test run. This way, you know immediately if something broke.

4. Test in a clone of the production environment. It does no good to test your software under different conditions from those that it will run in production; doing so is a recipe for disaster when you deploy. Never hear someone say “But it worked on my machine!” again.

5. Make all versions easily accessible. Despite best efforts, production releases still will break, so you need an easy way to re-deploy a prior version. Then, you’ll want to compare the working and broken versions to figure out what went wrong. To do this, you’ll need a reference copy of past releases.

6. Have an audit trail (that is, a blame list). This helps you not just in the source control of who made this commit, but who deployed the commit as well. This can provide rationale as well as potential fixes.

7. Automate site deployment. In order to tolerate small, frequent releases, pushing a release needs to be an automated process so it’s very quick and easy. If it’s a big chore to push one release, the whole process falls apart.

8. Measure results and iterate rapidly. Are the changes helping? Is the software faster?  Did the usability enhancement yield more sales?  If it’s not, you can iterate again.
