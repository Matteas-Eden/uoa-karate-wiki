<template>
  <div class="footer-navigation">
    <h2 class="footer-navigation-title">Wiki Pages</h2>
    <div
      v-for="(cat, i) in footerCategories"
      class="categories"
    >
      <div class="category-title">
        <router-link
          v-if="footerCategoriesObj[cat] && footerCategoriesObj[cat].path"
          :to="footerCategoriesObj[cat].path"
        >
          {{cat | categoryName}}
        </router-link>
        <span v-else>
          {{cat | categoryName}}
        </span>
      </div>
      <div class="pages">
        <router-link
          v-for="page in footerCategoriesObj[cat].pages"
          class="page-link"
          :to="page.path"
        >
          {{page.title}}
        </router-link>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  filters: {
    categoryName(name) {
      switch (name) {
        case "important":
          return "Important Pages";
        default:
          return name;
      }
    },
  },
  computed: {
    trimmedPageObjs() {
      let trimmedPageObjs = this.$site.pages
        .filter(
          (page) => !page.frontmatter.isDraft && !page.frontmatter.noFooter
        )
        .map((page) => {
          return {
            categories: page.frontmatter && page.frontmatter.categories,
            title: page.title,
            path: page.path,
            categoryParent: page.frontmatter.isCategoryParent,
          };
        });

      return trimmedPageObjs;
    },
    footerCategoriesObj() {
      // we add Important here so it's the first category to show in the footer
      const footerCategoriesObj = { important: {} };
      const miscPages = { pages: [] };
      const ignoredCategories = ["hojo undo", "kobudo", "yakusoku kumite"];

      this.trimmedPageObjs.forEach((page) => {
        if (page.categories && page.categories.length) {
          page.categories.forEach((cat) => {
            cat = cat.toLowerCase();

            // We don't want to display these categories right now
            if (ignoredCategories.includes(cat)) {
              // Put the page in misc if we don't want to display the category
              miscPages.pages.push(page);
            } else {
              if (!page.categoryParent) {
                if (
                  footerCategoriesObj[cat] &&
                  footerCategoriesObj[cat].pages
                ) {
                  footerCategoriesObj[cat].pages.push(page);
                } else {
                  footerCategoriesObj[cat] = { pages: [page] };
                }
              }

              if (page.categoryParent) {
                if (!footerCategoriesObj[cat]) {
                  footerCategoriesObj[cat] = { pages: [] };
                }
                footerCategoriesObj[cat].path = page.path;
              }
            }
          });
        } else {
          // If the page doesn't have a category, throw it in misc
          miscPages.pages.push(page);
        }
      });
      footerCategoriesObj.misc = miscPages;
      return footerCategoriesObj;
    },
    footerCategories() {
      return Object.keys(this.footerCategoriesObj);
    },
  },
};
</script>

<style scoped>
.footer-navigation {
  --border-colour: #ccc;
  border: 1px solid var(--border-colour);
}

.footer-navigation-title {
  font-size: 18px;
  text-align: center;
  padding: 10px 0;
  margin: 0;
  border: none;
}

.categories {
  display: flex;
  border-top: 1px solid var(--border-colour);
}

.category-title {
  padding: 10px;
  flex: 0 0 100px;
  border-right: 1px solid var(--border-colour);
  text-align: right;
  text-transform: capitalize;
}

.pages {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  padding: 10px;
}
</style>