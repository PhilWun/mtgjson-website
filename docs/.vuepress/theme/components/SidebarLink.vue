<script>
import { isActive, hashRE, groupHeaders } from "../util";

export default {
  functional: true,
  props: {
    item: {
      required: true,
      type: Object,
    },
    sidebarDepth: {
      default: 0,
      type: Number,
    },
  },
  render(
    h,
    {
      parent: { $page, $site, $route, $themeConfig, $themeLocaleConfig },
      props: { item, sidebarDepth },
    }
  ) {
    // use custom active class matching logic
    // due to edge case of paths ending with / + hash
    const selfActive = isActive($route, item.path);
    // for sidebar: auto pages, a hash link should be active if one of its child
    // matches
    const active =
      item.type === "auto"
        ? selfActive ||
          item.children.some((c) =>
            isActive($route, item.basePath + "#" + c.slug)
          )
        : selfActive;
    const link = renderLink(h, item.path, item.title || item.path, active);

    const configDepth =
      $page.frontmatter.sidebarDepth ||
      sidebarDepth ||
      $themeLocaleConfig.sidebarDepth ||
      $themeConfig.sidebarDepth;

    const maxDepth = configDepth === null ? 1 : configDepth;

    const displayAllHeaders =
      $themeLocaleConfig.displayAllHeaders || $themeConfig.displayAllHeaders;

    if (item.type === "auto") {
      return [
        link,
        renderChildren(h, item.children, item.basePath, $route, maxDepth),
      ];
    } else if (
      (active || displayAllHeaders) &&
      item.headers &&
      !hashRE.test(item.path)
    ) {
      const children = groupHeaders(item.headers);
      return [link, renderChildren(h, children, item.path, $route, maxDepth)];
    } else {
      return link;
    }
  },
};

function renderLink(h, to, text, active) {
  return h(
    "router-link",
    {
      props: {
        to,
        activeClass: "",
        exactActiveClass: "",
      },
      class: {
        active,
        "sidebar-link": true,
      },
    },
    text
  );
}

function renderChildren(h, children, path, route, maxDepth, depth = 1) {
  if (!children || depth > maxDepth) {
    return null;
  }
  return h(
    "ul",
    { class: "sidebar-sub-headers" },
    children.map((c) => {
      const active = isActive(route, path + "#" + c.slug);
      return h("li", { class: "sidebar-sub-header" }, [
        renderLink(h, path + "#" + c.slug, c.title, active),
        renderChildren(h, c.children, path, route, maxDepth, depth + 1),
      ]);
    })
  );
}
</script>

<style lang="scss">
a.sidebar-link {
  font-size: 16px;
  font-weight: bold;
  display: inline-block;
  color: var(--text-color);
  line-height: 2;
  width: 100%;
  box-sizing: border-box;

  &.active {
    position: relative;
    color: var(--accent-color);
    font-weight: bold;
  }

  &:hover {
    color: var(--accent-color);
  }
}
</style>
