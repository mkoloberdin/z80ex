cmake_minimum_required (VERSION 2.8)
project (z80ex)

#ALL_CFLAGS := -fPIC -fno-common -ansi -pedantic -Wall -pipe -O2 -I. -I./include
if (${CMAKE_COMPILER_IS_GNUCC})
    set (CMAKE_C_FLAGS "-fPIC -fno-common -ansi -pedantic -Wall -pipe -O2")
endif ()

include_directories(BEFORE . include)

include (TestBigEndian)
test_big_endian(BIG_ENDIAN)
#endianness (one of: WORDS_LITTLE_ENDIAN, WORDS_BIG_ENDIAN)
if (BIG_ENDIAN)
    set (ENDIANNESS WORDS_BIG_ENDIAN)
else ()
    set (ENDIANNESS WORDS_LITTLE_ENDIAN)
endif ()

#fast and rough opcode step emulation mode (0 - off, 1 - on)
set (OPSTEP_FAST_AND_ROUGH 0)

set (EMU, libz80ex)
set (DASM, libz80ex_dasm)
set (API_REVISION 1)
set (VERSION_MAJOR 1)
set (VERSION_MINOR 18)
set (RELEASE_TYPE "")
set (VERSION_STR "${API_REVISION}.${VERSION_MAJOR}.${VERSION_MINOR}${RELEASE_TYPE}")

add_definitions (-D${ENDIANNESS} -DZ80EX_VERSION_STR=${VERSION_STR} -DZ80EX_API_REVISION=${API_REVISION} -DZ80EX_VERSION_MAJOR=${VERSION_MAJOR} -DZ80EX_VERSION_MINOR=${VERSION_MINOR} -DZ80EX_RELEASE_TYPE=${RELEASE_TYPE})

if (${OPSTEP_FAST_AND_ROUGH})
    add_definitions (-DZ80EX_OPSTEP_FAST_AND_ROUGH)
endif ()

#c_files := z80ex.c z80ex_dasm.c

#%.o : %.c
#	${CC} ${ALL_CFLAGS} ${CFLAGS} -c -o $@ $<	

#.PHONY : all
#all:: static shared

#set (z80ex_sources
#    ptables.c
#    opcodes/opcodes_base.c
#    opcodes/opcodes_dd.c
#    opcodes/opcodes_fd.c
#    opcodes/opcodes_cb.c
#    opcodes/opcodes_ed.c
#    opcodes/opcodes_ddcb.c
#    opcodes/opcodes_fdcb.c
#)
set (z80ex_sources z80ex.c)

add_library (z80ex SHARED ${z80ex_sources})
add_library (z80ex-static STATIC ${z80ex_sources})
set_target_properties (z80ex-static PROPERTIES OUTPUT_NAME z80ex)
# Affects Win32 only: avoid dynamic/static *.lib files naming conflict 
set_target_properties (z80ex-static PROPERTIES PREFIX "lib")

#set (z80ex_dasm_sources opcodes/opcodes_dasm.c)
set (z80ex_dasm_sources z80ex_dasm.c)
add_library (z80ex_dasm SHARED ${z80ex_dasm_sources})
add_library (z80ex_dasm-static STATIC ${z80ex_dasm_sources})
set_target_properties (z80ex_dasm-static PROPERTIES OUTPUT_NAME z80ex_dasm)
# Affects Win32 only: avoid dynamic/static *.lib files naming conflict 
set_target_properties (z80ex_dasm-static PROPERTIES PREFIX "lib")

set_target_properties(z80ex z80ex-static z80ex_dasm z80ex_dasm-static
    PROPERTIES VERSION ${VERSION_STR} SOVERSION ${API_REVISION}
)

if ("${CMAKE_C_IMPLICIT_LINK_DIRECTORIES}" MATCHES "lib64")
    set (LIB_DIR "lib64")
else ()
    set (LIB_DIR "lib")
endif ()
install (TARGETS z80ex z80ex-static z80ex_dasm z80ex_dasm-static LIBRARY DESTINATION ${LIB_DIR} ARCHIVE DESTINATION ${LIB_DIR})
install (DIRECTORY include/ DESTINATION include PATTERN "*.h" )

#clean:
#	rm -f *.o
#	rm -f ./lib/*
#	rm -rf ./z80ex-${VERSION_STR}.tar.gz
#
#static: z80ex.o z80ex_dasm.o
#	ar rs ./lib/${EMU}.a z80ex.o
#	ar rs ./lib/${DASM}.a z80ex_dasm.o
	
#shared: z80ex.o z80ex_dasm.o
#ifeq (${OS},Darwin)
#	${LINKER} -dynamiclib -compatibility_version ${API_REVISION} -current_version ${VERSION_STR} -install_name ${INSTALL_PREFIX}/lib/${EMU}.${API_REVISION}.dylib -o ./lib/${EMU}.${VERSION_STR}.dylib z80ex.o
#	${LINKER} -dynamiclib -compatibility_version ${API_REVISION} -current_version ${VERSION_STR} -install_name ${INSTALL_PREFIX}/lib/${DASM}.${API_REVISION}.dylib -o ./lib/${DASM}.${VERSION_STR}.dylib z80ex_dasm.o
#else
#	${LINKER} -shared -Wl,-soname,${EMU}.so.${API_REVISION} -o ./lib/${EMU}.so.${VERSION_STR} z80ex.o
#	${LINKER} -shared -Wl,-soname,${DASM}.so.${API_REVISION} -o ./lib/${DASM}.so.${VERSION_STR} z80ex_dasm.o	
#endif
#	
#install:
#	install -d ${INSTALL_PREFIX}/lib
#	install ./lib/* ${INSTALL_PREFIX}/lib
#	install -d ${INSTALL_PREFIX}/include/z80ex
#	install -m 0664 ./include/* ${INSTALL_PREFIX}/include/z80ex
#ifeq (${OS},Darwin)
#	ln -sf ${EMU}.${VERSION_STR}.dylib ${INSTALL_PREFIX}/lib/${EMU}.${API_REVISION}.dylib
#	ln -sf ${EMU}.${VERSION_STR}.dylib ${INSTALL_PREFIX}/lib/${EMU}.dylib
#	ln -sf ${DASM}.${VERSION_STR}.dylib ${INSTALL_PREFIX}/lib/${DASM}.${API_REVISION}.dylib
#	ln -sf ${DASM}.${VERSION_STR}.dylib ${INSTALL_PREFIX}/lib/${DASM}.dylib	
#else
#	ln -sf ${EMU}.so.${VERSION_STR} ${INSTALL_PREFIX}/lib/${EMU}.so.${API_REVISION}
#	ln -sf ${EMU}.so.${VERSION_STR} ${INSTALL_PREFIX}/lib/${EMU}.so
#	ln -sf ${DASM}.so.${VERSION_STR} ${INSTALL_PREFIX}/lib/${DASM}.so.${API_REVISION}
#	ln -sf ${DASM}.so.${VERSION_STR} ${INSTALL_PREFIX}/lib/${DASM}.so		
#endif	
#
#dist: clean
#	rm -rf ./${PROJ}-${VERSION_STR}${RELEASE_TYPE}
#	ln -s ./ ./${PROJ}-${VERSION_STR}${RELEASE_TYPE}
#	tar --exclude z80ex.geany --exclude obsolete --exclude ${PROJ}-${VERSION_STR}${RELEASE_TYPE}/${PROJ}-${VERSION_STR}${RELEASE_TYPE} --exclude ${PROJ}-${VERSION_STR}${RELEASE_TYPE}/${PROJ}-${VERSION_STR}${RELEASE_TYPE}.tar.gz -hcf - ./${PROJ}-${VERSION_STR}${RELEASE_TYPE}/ | gzip -f9 > ${PROJ}-${VERSION_STR}${RELEASE_TYPE}.tar.gz
#	rm -rf ./${PROJ}-${VERSION_STR}${RELEASE_TYPE}

#EOF
