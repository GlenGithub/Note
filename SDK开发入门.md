һ�� SDK���
SDK :Software Development Kit ����������߰���������������룬һ���Լ���api���ĵ������������ߵ���ʽ���֣�Ϊ�������ṩ������ĵ��ýӿ�

�� ģ����SDK
����������רҵ��ǿ��SDK������֧�������ͣ���ͼ��IM��ӵ�ж������ܵ�SDK��
�� ������SDK
���ܵ�һ�Ĺ�����SDK���������ݿ⣬���磬�ļ�IO
�� UI�ؼ���SDK
����ؼ���SDK

��������SDK���е�
�� �����ظ����뿪��
�� ���ٴ������
�� ��Լ�����ɱ�

��������SDK����
�� ����Library
�� jar��������ã������build/libs��
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

�� �����������������
def getVersionNameRelease(){
  return 'app_1.0_'+new Date().format('YYYYMMdd,TimeZone.getDefault()+'.jar')
}

def getVersionNameDebug(){
  return 'app_1.0_'+new Date().format('YYYYMMdd,TimeZone.getDefault()+'.debug.jar')
}








