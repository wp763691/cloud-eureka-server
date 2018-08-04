env.BUILD_VERSION               = null
env.CURRENT_STAGE               = null
env.FAILED_REASON               = null
env.CURRENT_BRANCH              = null
env.CURRENT_WORKSPACE_DIR       = null

node {
    def workspace = env.WORKSPACE
    env.CURRENT_WORKSPACE_DIR = "${workspace}@script"
    echo env.CURRENT_WORKSPACE_DIR

    env.BUILD_VERSION = version()
    echo env.BUILD_VERSION

    //截获pom.xml里面的版本最后6位字符，用来判断是编译什么环境
    tempstr = env.BUILD_VERSION.substring(6).toLowerCase()
    if (tempstr.endsWith('snapshot')) {
        //开发分支
        env.CURRENT_BRANCH = 'master'
    } else if (tempstr.endsWith('release')) {
        //发布分支
        env.CURRENT_BRANCH = 'release'
    }

    stage('Check out') {
        echo "Current branch is ${env.CURRENT_BRANCH}"

        try {
            git url: "https://github.com/wp763691/cloud-eureka-server.git", credentialsId: "github", branch: "${env.CURRENT_BRANCH}"
        } catch (e) {
            env.CURRENT_STAGE = '拉取GIT代码'
            env.FAILED_REASON = e
            failedNotification()
            throw e
        }
    }

    stage('Compile') {
        try {
            sh 'mvn clean package'
        } catch (e) {
            env.CURRENT_STAGE = '编译代码'
            env.FAILED_REASON = e
            failedNotification()
            throw e
        }
    }

    stage('Build docker image and push to registry') {
        try {
            sh 'rm -rf *.jar'
            sh 'cp target/*.jar app.jar'
            docker.withRegistry("http://192.168.0.201:5000") {
                def customImage = docker.build("192.168.0.201:5000/cloud_microservice/cloud-eureka-server:${env.BUILD_VERSION}")
                customImage.push()
            }
        } catch (e) {
            env.CURRENT_STAGE = '打包镜像 & 推送到仓库'
            env.FAILED_REASON = e
            failedNotification()
            throw e
        }
    }
}

def version() {
    def workspace_dir = env.CURRENT_WORKSPACE_DIR
    echo workspace_dir
    def matcher = readFile(workspace_dir+'/pom.xml') =~ '<version>(.+)</version>'
    matcher ? matcher[0][1] : null
}

def failedNotification() {
    emailext(
            subject: "构建失败: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]'",
            from: "763691@163.com",
            to: "763691@163.com",
            body: """<p>构建失败: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]':</p>
                <p>失败阶段: [${env.CURRENT_STAGE}]</p>
                <p>失败异常: ${env.FAILED_REASON}]</p>
                <p>点击查看构建详情 "<a href="${env.BUILD_URL}/console">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""
    )
}

def successulNotification() {
    emailext(
            subject: "构建成功: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]'",
            from: "763691@163.com",
            to: "763691@163.com",
            body: """<p>构建成功: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]':</p>
                <p>点击查看构建详情 "<a href="${env.BUILD_URL}/console">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""
    )
}

def releassEnvSuccessulNotification() {
    emailext(
            subject: "构建成功: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]'",
            from: "763691@163.com",
            to: "763691@163.com",
            body: """<p>构建成功: Job '${env.JOB_NAME} 版本 [${env.BUILD_VERSION}]':</p>
                <p>点击查看构建详情 "<a href="${env.BUILD_URL}/console">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""
    )
}