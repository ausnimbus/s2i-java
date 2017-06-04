/**
* NOTE: THIS JENKINSFILE IS GENERATED VIA "./hack/run update"
*
* DO NOT EDIT IT DIRECTLY.
*/
node {
        def variants = "alpine".split(',');
        for (int v = 0; v < variants.length; v++) {

                def versions = "3".split(',');
                for (int i = 0; i < versions.length; i++) {

                  if (variants[v] == "default") {
                    variant = ""
                    tag = versions[i]
                  } else {
                    variant = variants[v]
                    tag = versions[i] + "-" + variant
                  }


                        try {
                                stage("Build (Maven-${tag})") {
                                        openshift.withCluster() {
        openshift.apply([
                                "apiVersion" : "v1",
                                "items" : [
                                        [
                                                "apiVersion" : "v1",
                                                "kind" : "ImageStream",
                                                "metadata" : [
                                                        "name" : "maven",
                                                        "labels" : [
                                                                "builder" : "s2i-maven"
                                                        ]
                                                ],
                                                "spec" : [
                                                        "tags" : [
                                                                [
                                                                        "name" : "${tag}",
                                                                        "from" : [
                                                                                "kind" : "DockerImage",
                                                                                "name" : "maven:${versions[i]}-jdk-8-alpine",
                                                                        ],
                                                                        "referencePolicy" : [
                                                                                "type" : "Source"
                                                                        ]
                                                                ]
                                                        ]
                                                ]
                                        ],
                                        [
                                                "apiVersion" : "v1",
                                                "kind" : "ImageStream",
                                                "metadata" : [
                                                        "name" : "s2i-maven",
                                                        "labels" : [
                                                                "builder" : "s2i-maven"
                                                        ]
                                                ]
                                        ]
                                ],
                                "kind" : "List"
                        ])
        openshift.apply([
                                "apiVersion" : "v1",
                                "kind" : "BuildConfig",
                                "metadata" : [
                                        "name" : "s2i-maven-${tag}",
                                        "labels" : [
                                                "builder" : "s2i-maven"
                                        ]
                                ],
                                "spec" : [
                                        "output" : [
                                                "to" : [
                                                        "kind" : "ImageStreamTag",
                                                        "name" : "s2i-maven:${tag}"
                                                ]
                                        ],
                                        "runPolicy" : "Serial",
                                        "resources" : [
                                            "limits" : [
                                                "memory" : "2Gi"
                                            ]
                                        ],
                                        "source" : [
                                                "git" : [
                                                        "uri" : "https://github.com/ausnimbus/s2i-maven"
                                                ],
                                                "type" : "Git"
                                        ],
                                        "strategy" : [
                                                "dockerStrategy" : [
                                                        "dockerfilePath" : "versions/${versions[i]}/${variant}/Dockerfile",
                                                        "from" : [
                                                                "kind" : "ImageStreamTag",
                                                                "name" : "maven:${tag}"
                                                        ]
                                                ],
                                                "type" : "Docker"
                                        ]
                                ]
                        ])
        echo "Created s2i-maven:${tag} objects"
        /**
        * TODO: Replace the sleep with import-image
        * openshift.importImage("maven:${tag}")
        */
        sleep 60

        echo "==============================="
        echo "Starting build s2i-maven-${tag}"
        echo "==============================="
        def builds = openshift.startBuild("s2i-maven-${tag}");

        timeout(20) {
                builds.untilEach(1) {
                        return it.object().status.phase == "Complete"
                }
        }
        echo "Finished build ${builds.names()}"
        builds.logs()
}

                                }
                                stage("Test (Maven-${tag})") {
                                        openshift.withCluster() {
        echo "==============================="
        echo "Starting test application"
        echo "==============================="

        def testApp = openshift.newApp("https://github.com/ausnimbus/maven-ex", "--image-stream=s2i-maven:${tag}", "-l app=maven-ex");
        echo "new-app created ${testApp.count()} objects named: ${testApp.names()}"
        testApp.describe()

        def testAppBC = testApp.narrow("bc");
        def testAppBuilds = testAppBC.related("builds");
        echo "Waiting for ${testAppBuilds.names()} to finish"
        timeout(10) {
                testAppBuilds.untilEach(1) {
                        return it.object().status.phase == "Complete"
                }
        }
        echo "Finished ${testAppBuilds.names()}"

        def testAppDC = testApp.narrow("dc");
        echo "Waiting for ${testAppDC.names()} to start"
        timeout(10) {
                testAppDC.untilEach(1) {
                        return it.object().status.availableReplicas >= 1
                }
        }
        echo "${testAppDC.names()} is ready"

        def testAppService = testApp.narrow("svc");
        def testAppHost = testAppService.object().spec.clusterIP;
        def testAppPort = testAppService.object().spec.ports[0].port;

        sleep 60
        echo "Testing endpoint ${testAppHost}:${testAppPort}"
        sh "curl -o /dev/null $testAppHost:$testAppPort"
}

                                }
                                stage("Stage (Maven-${tag})") {
                                        openshift.withCluster() {
        echo "==============================="
        echo "Tag new image into staging"
        echo "==============================="

        openshift.tag("ausnimbus-ci/s2i-maven:${tag}", "ausnimbus/s2i-maven:${tag}")
}

                                }
                        } finally {
                                openshift.withCluster() {
                                        echo "Deleting test resources maven-ex"
                                        openshift.selector("dc", [app: "maven-ex"]).delete()
                                        openshift.selector("bc", [app: "maven-ex"]).delete()
                                        openshift.selector("svc", [app: "maven-ex"]).delete()
                                        openshift.selector("is", [app: "maven-ex"]).delete()
                                        openshift.selector("pods", [app: "maven-ex"]).delete()
                                        openshift.selector("routes", [app: "maven-ex"]).delete()
                                }
                        }

                }
        }
}
