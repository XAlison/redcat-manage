<template>
  <iframe :src="this.menuSrc" width="100%" height="100%" frameborder="0"></iframe>
</template>
<script>
export default {
  name: 'iframe-component',
  props: ['iframeSrc'],
  data () {
    let menuSrc = ''
    switch (this.iframeSrc) {
      case 'employeeEdit':
        menuSrc = '/web-static/html/organization/employeeEdit.html' + this.$route.fullPath.substring(20)
        break
      case 'employeeList':
        menuSrc = '/web-static/html/organization/employeeList.html'
        break
      case 'organization':
        menuSrc = '/web-static/html/organization/organization.html'
        break
      case 'rolePermissions':
        menuSrc = '/web-static/html/role/roleEdit.html'
        break
      case 'securityPassword':
        menuSrc = '/web-static/html/report/reportSecurity.html'
        break
    }
    return {
      menuSrc
    }
  }

}
</script>
