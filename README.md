====================[ Build | tp3_bottin_im3210 | Debug ]=======================
/opt/clion-2024.2.1/bin/cmake/linux/aarch64/bin/cmake --build /home/etudiant/CLionProjects/tp3-bottin-im3210/cmake-build-debug --target tp3_bottin_im3210 -j 1
[2/2] Linking CXX executable tp3_bottin_im3210
FAILED: tp3_bottin_im3210 
: && /usr/bin/c++ -g  CMakeFiles/tp3_bottin_im3210.dir/ContratException.cpp.o CMakeFiles/tp3_bottin_im3210.dir/Bottin.cpp.o -o tp3_bottin_im3210   && :
/usr/bin/ld: /usr/lib/gcc/aarch64-linux-gnu/11/../../../aarch64-linux-gnu/Scrt1.o: in function `_start':
(.text+0x1c): undefined reference to `main'
/usr/bin/ld: (.text+0x20): undefined reference to `main'
collect2: error: ld returned 1 exit status
ninja: build stopped: subcommand failed.

