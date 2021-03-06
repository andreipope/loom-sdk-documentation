<script>
import { isActive, hashRE, groupHeaders } from '../util'
export default {
  functional: true,
  props: ['item', 'sidebarDepth'],
  render(
    h,
    {
      parent: { $page, $site, $route, $themeConfig, $themeLocaleConfig, $t },
      props: { item, sidebarDepth }
    }
  ) {
    const selfActive = isActive($route, item.path)
    const active =
      item.type === 'auto'
        ? selfActive || item.children.some(c => isActive($route, item.basePath + '#' + c.slug))
        : selfActive
    const link = renderLink(h, item.path, $t(item.title || item.path), active)
    const configDepth =
      $page.frontmatter.sidebarDepth ||
      sidebarDepth ||
      $themeLocaleConfig.sidebarDepth ||
      $themeConfig.sidebarDepth
    const maxDepth = configDepth == null ? 1 : configDepth
    const displayAllHeaders = $themeLocaleConfig.displayAllHeaders || $themeConfig.displayAllHeaders
    if (item.type === 'auto') {
      return [link, renderChildren(h, item.children, item.basePath, $route, maxDepth)]
    } else if ((active || displayAllHeaders) && item.headers && !hashRE.test(item.path)) {
      const children = groupHeaders(item.headers)
      return [link, renderChildren(h, children, item.path, $route, maxDepth)]
    } else {
      return link
    }
  }
}

function renderLink(h, to, text, active) {
  return h(
    'router-link',
    {
      props: {
        to,
        activeClass: '',
        exactActiveClass: ''
      },
      class: {
        active,
        'sidebar-link': true
      }
    },
    text
  )
}

function renderChildren(h, children, path, route, maxDepth, depth = 1) {
  if (!children || depth > maxDepth) return null
  return h(
    'ul',
    { class: 'sidebar-sub-headers' },
    children.map(c => {
      const active = isActive(route, path + '#' + c.slug)
      return h('li', { class: 'sidebar-sub-header' }, [
        renderLink(h, path + '#' + c.slug, c.title, active),
        renderChildren(h, c.children, path, route, maxDepth, depth + 1)
      ])
    })
  )
}
</script>

<style lang="stylus">
.sidebar .sidebar-sub-headers
  padding-left 1rem
  font-size 1.1em
a.sidebar-link
  font-size 1.1em
  font-weight 500
  display inline-block
  border-left 0.25rem solid transparent
  padding 0.35rem 1rem 0.35rem 1.25rem
  width: 100%
  box-sizing: border-box
  color $sidebarLinkChildrenColor
  &:hover
    color $accentColor
  &.active
    font-weight 600
    color $accentColor
    border-left-color $accentColor
  .sidebar-group &
    padding-left 2rem
  .sidebar-sub-headers &
    padding-top 0.25rem
    padding-bottom 0.25rem
    border-left none
    &.active
      font-weight 500
</style>
