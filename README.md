---
# layout: home
# title: Home / Announcements
nav_exclude: true
# permalink: /:path/
# seo:
#   type: Course
#   name: Just the Class
---


# How to Update Decal site

Only a few people have access to update the website now as the repo is hosted on the Extended Reality @ Berkeley GitHub organization.

Let me know if you need to be added to the organization to update the website.

---

1. After previewing your site locally to make sure everything looks good, you will need to build the site by running `JEKYLL_ENV=production bundle exec jekyll b`. This will build to the directory `_site`.

2. Copy all the content inside `_site` (don't copy the `_site` folder itself) into the `decal` folder in the website GitHub repository. Commit & push to the website repo.

3. To modify the live website, you will also have to pull the changes from within OCF, our hosting platform. Instructions to do that are on the website repo.


## Building and previewing your site locally

[https://just-the-docs.github.io/just-the-docs-template/](https://just-the-docs.github.io/just-the-docs-template/)

Assuming [Jekyll] and [Bundler] are installed on your computer:

1.  Change your working directory to the root directory of your site.

2.  Run `bundle install`.

3.  Run `bundle exec jekyll serve` to build your site and preview it at `localhost:4000`.

    The built site is stored in the directory `_site`.


### Template Credits: Just the Class

Just the Class is a GitHub Pages template developed for the purpose of quickly deploying course websites. In addition to serving plain web pages and files, it provides a boilerplate for:

- [announcements](announcements.md),
- a [course calendar](calendar.md),
- a [staff](staff.md) page,
- and a weekly [schedule](schedule.md).

Just the Class is a template that extends the popular [Just the Docs](https://github.com/just-the-docs/just-the-docs) theme, which provides a robust and thoroughly-tested foundation for your website. Just the Docs include features such as:

- automatic [navigation structure](https://just-the-docs.github.io/just-the-docs/docs/navigation-structure/),
- instant, full-text [search](https://just-the-docs.github.io/just-the-docs/docs/search/) and page indexing,
- and a set of [UI components](https://just-the-docs.github.io/just-the-docs/docs/ui-components) and authoring [utilities](https://just-the-docs.github.io/just-the-docs/docs/utilities).