### conan安装
* pip
* 官网

### conan profiles
* ``conan config home`` 显示conan配置路径
* ``conan profile detect`` 检测conan profile(没有会生成一个默认的，根据本机的环境)

### conanfile
* ``[requires]`` 依赖的包们
* ``[generators]`` 生成(cmake配合)
  * ``CMakeDeps``
  * ``CMakeToolchain``  


### conan install
* conan install <conanfile的路径> 
  * --profile=<profile的路径/默认profiles下的文件名>
  * -s build_type=<构建类型>
  * --output-folder=<生成的文件路径>
  * -of=<conan生成的相关文件存放目录> 
  * --build=missing(如果没有下载依赖的库进行下载)  