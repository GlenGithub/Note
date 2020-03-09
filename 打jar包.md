task makejar(type:Copy){
    //删除存在的
    delete 'build/libs/mdm.jar'
    //设置拷贝的文件
    from('build/intermediates/packaged-classes/release/')
    //打进jar包后的文件目录
    into('build/libs/')
    include('classes.jar')
    //重命名
    rename('classes.jar','mdm.jar')
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
//    exclude '**/output/**'
    classifier = 'sources'
}

makejar.dependsOn(build)