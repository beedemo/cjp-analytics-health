#!groovy
import groovy.json.*

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

stage('Update ES Health') {
  def sysEnv = System.getenv()
  node('docker') {
    def url = "${sysEnv.ES_HOST}/_cluster/health"
    def resp = httpRequest acceptType: 'APPLICATION_JSON', httpMode: 'GET', authentication: "$esHttpReqAuthId", url: "$url", validResponseCodes: '100:500'
    def respObj = jsonParse(resp.content)
    println "es resp: ${respObj}"
    
    //set up path for index
    def dateSuffix = new Date().format( 'yyyy-MM' )
    def esIndex = "es-health-$dateSuffix"
    def indexUrl = "${sysEnv.ES_HOST}/${esIndex}/health/"
    def indexResp = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'PUT', requestBody: respObj, authentication: "$esHttpReqAuthId", url: "$url", validResponseCodes: '100:500'
    def indeRrespObj = jsonParse(indexResp.content)
    println "es health index resp: ${indeRrespObj}"
  }
}
