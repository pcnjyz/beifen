#Termux

pkg update && pkg install make clang build-essential

pkg install libzip

make clean && TERMUX=1 make

./qdl


su

./qdl /storage/emulated/0/images/DevprgProgrammer2.elf /storage/emulated/0/images/rawprogram*.xml /storage/emulated/0/images/patch*.xml
