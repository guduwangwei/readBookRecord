for 循环标准写法

 int a = 0 ;

 for(; a< 5;a++){

….

}

* 函数指针  

  ~~~~c
  #include <stdio.h>

  void message() {
      printf("%s", "message调用");
  }

  void invakeMesage(void (*func)()) {
      func();
  }


  int main() {
      //函数指针
  //    void (*FunP)()= message;
  //    FunP();
      invakeMesage(message);
      return 0;
  }

  ~~~~

  ​