一、 SDK简介
SDK :Software Development Kit 软件开发工具包，属于组件化范畴，一般以集合api和文档，范例，工具的形式出现，为开发者提供工具类的调用接口

■ 模块类SDK
功能完整，专业性强的SDK，类似支付，推送，地图，IM等拥有独立功能的SDK。
■ 工具类SDK
功能单一的工具类SDK，比如数据库，网络，文件IO
■ UI控件类SDK
界面控件类SDK

二、开发SDK的有点
■ 减少重复代码开发
■ 减少代码耦合
■ 节约开发成本

三、开发SDK步骤
■ 创建Library
■ jar包打包配置（存放于build/libs）
import proguard.gradle.ProGuardTask
def jarPath = 'build/libs'
task makeJarMinif(type: ProGuardTask,dependsOn: "build"){
    delete jarPath
	injars 'build/intermediates/packaged-classes/release/classes.jar'
	outjars jarPath
	configuration 'proguard-rules.pro'
}

task makeJarRelease(type: Jar,dependsOn: ['assembleRelease']){
    archiveName = getVersionNameRelease()
	from('build/intermediates/classes/release')
	destinationDir = file('build/libs')
	include('com/glen/**/*.class')
}

task makeJarDebug(type: Jar,dependsOn: ['assembleDebug']){
    archiveName = getVersionNameDebug()
	from('build/intermediates/classes/debug')
	destinationDir = file('build/libs')
	include('com/glen/**/*.class')
}

task sourcesJar(type: Jar){
    from android.sourceSets.main.java.srcDirs
	exclude '**/output/**'
	classifier = 'sources'
}

■ 打包过程中命名配置
def getVersionNameRelease(){
  return 'app_1.0_'+new Date().format('YYYYMMdd,TimeZone.getDefault()+'.jar')
}

def getVersionNameDebug(){
  return 'app_1.0_'+new Date().format('YYYYMMdd,TimeZone.getDefault()+'.debug.jar')
}








