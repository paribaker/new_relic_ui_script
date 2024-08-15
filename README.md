This is a script that hosts the new-relic minified JS to be able to add to your app, here is a sample for setting thing sup in vue. 

1. Create a new plugin in `plugins/new-relic-plugin.js`
   
```
const NewRelicUIMonitoringPlugin = {
  install(app, options) {
    if (window.NREUM) {
      console.info('Clearing existing NR')
      window.NREUM = undefined
    }
    window.NREUM = options
    const script = document.createElement('script')
    script.setAttribute('type', 'text/javascript')
    script.setAttribute('crossorigin', 'anonymous')
    script.setAttribute(
      'src',
      'https://cdn.jsdelivr.net/gh/paribaker/new_relic_ui_script@latest/newrelic_script.js',
    )
    document.head.appendChild(script)
  },
}

export default NewRelicUIMonitoringPlugin

```


Add it to your `main.js|ts`

```

import NewRelicUIMonitoringPlugin from './plugins/new-relic-plugin'

/**
Since this can be used for multiple envs with only one new relic app
**/

const PROD_ENDPOINTS = [
  'prod-app.herokuapp.com',
  'app.someapp.com',
]
const LOCATION = `${window.location.protocol}//${window.location.host}`
let IS_PROD = false

PROD_ENDPOINTS.forEach((endpoint) => {
  const regex = new RegExp('\\b' + endpoint.replace(/\./g, '\\.') + '\\b', 'i')
  if (regex.test(LOCATION)) {
    IS_PROD = true
    return
  }
})

const PROD_LOADER_CONFIG = {
  accountID: '<ACCOUNT_ID>',
  trustKey: '<TRUST_ID>',
  agentID: '<AGENT_ID>',
  licenseKey: '<LICENSE_KEY>',
  applicationID: '<APP_ID>',
}
const STAGING_LOADER_CONFIG = {
  accountID: '<ACCOUNT_ID>',
  trustKey: '<TRUST_ID>',
  agentID: '<AGENT_ID>',
  licenseKey: '<LICENSE_KEY>',
  applicationID: '<APP_ID>',
}

const config = {
  init: {
    distributed_tracing: { enabled: true },
    privacy: { cookies_enabled: true },
    ajax: { deny_list: ['bam.nr-data.net'] },
  },
  loader_config: {
    accountID: IS_PROD ? PROD_LOADER_CONFIG.accountID : STAGING_LOADER_CONFIG.accountID,
    trustKey: IS_PROD ? PROD_LOADER_CONFIG.trustKey : STAGING_LOADER_CONFIG.trustKey,
    agentID: IS_PROD ? PROD_LOADER_CONFIG.agentID : STAGING_LOADER_CONFIG.agentID,
    licenseKey: IS_PROD ? PROD_LOADER_CONFIG.licenseKey : STAGING_LOADER_CONFIG.licenseKey,
    applicationID: IS_PROD ? PROD_LOADER_CONFIG.applicationID : STAGING_LOADER_CONFIG.applicationID,
  },
  info: {
    beacon: 'bam.nr-data.net',
    errorBeacon: 'bam.nr-data.net',
    licenseKey: IS_PROD ? PROD_LOADER_CONFIG.licenseKey : STAGING_LOADER_CONFIG.licenseKey,
    applicationID: IS_PROD ? PROD_LOADER_CONFIG.applicationID : STAGING_LOADER_CONFIG.applicationID,
    sa: 1,
  },
}
createApp(App)
  .use(NewRelicUIMonitoringPlugin, config)
  .mount('#app')
```
