cmake_minimum_required(VERSION 3.14)

set(CMAKE_VERBOSE_MAKEFILE ON)

project ("cmake-nasm-playground")

# #Set C++ language version
# set(CMAKE_CXX_STANDARD 17)
#Enable ASM provided by NASM
enable_language(ASM_NASM)

if(APPLE)
	#Add leading underscore when building macho64 object files
	string(APPEND CMAKE_ASM_NASM_FLAGS "--prefix _")
endif(APPLE)

# nasm -f win64 main.asm -o main.obj
# GoLink /entry:Start /console kernel32.dll main.obj

#If the goal is to interact with C++ code and assembly code, calling convention
#changes between platform, some adjustment on the assembly code is required.

#The following will pre-define a NASM macro "WIN32". This permit to conditionally assemble
#code in the following manner :
#  %ifdef WIN32
#     ;code for Windows ABI
#  %else
#     ;code for System V AMD64 ABI
#  %endif
if(WIN32)
    # nasm.exe -fwin64 -dWIN32=1 "CMakeFiles/cmake-nasm-playground.dir/main.asm.obj" -o cmake-nasm-playground.exe
    # nasm -f win64 main.asm -o main.obj
	string(APPEND CMAKE_ASM_NASM_FLAGS "-fwin64 -dWIN32=1")
   #  set(CMAKE_ASM_NASM_FLAGS "${CMAKE_ASM_NASM_FLAGS} -DWIN64")
endif(WIN32)

#Make a EXE with cpp and asm files
add_executable(${PROJECT_NAME} 
    "main.asm"
)

set(LDFLAGS "-nostdlib")

set(CMAKE_ASM_LINK_FLAGS "-Wl,-e,Start -Wl,--subsystem,console -lkernel32")
set(CMAKE_ASM_LINK_EXECUTABLE "${CMAKE_LINKER} <FLAGS> <CMAKE_ASM_LINK_FLAGS> <LINK_FLAGS> <OBJECTS> -o <TARGET> <LINK_LIBRARIES>")

set_target_properties(${ELF_TARGET} PROPERTIES LINK_FLAGS ${LDFLAGS})

message("CMAKE_ASM_LINK_EXECUTABLE ${CMAKE_ASM_LINK_EXECUTABLE}")
# SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE  ASM)
# SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES LINK_FLAGS )


# gcc -o main.exe -nostartfiles -Wl,-e,Start -Wl,--subsystem,console main.obj -lkernel32