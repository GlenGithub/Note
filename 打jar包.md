task makejar(type:Copy){
    //ɾ�����ڵ�
    delete 'build/libs/mdm.jar'
    //���ÿ������ļ�
    from('build/intermediates/packaged-classes/release/')
    //���jar������ļ�Ŀ¼
    into('build/libs/')
    include('classes.jar')
    //������
    rename('classes.jar','mdm.jar')
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
//    exclude '**/output/**'
    classifier = 'sources'
}

makejar.dependsOn(build)