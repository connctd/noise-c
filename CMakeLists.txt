idf_component_register()

set(NEWLIB_PATH ${IDF_PATH}/components/newlib/)

externalproject_add(noiseprotocol_build
    PREFIX ${COMPONENT_DIR}
    SOURCE_DIR ${COMPONENT_DIR}
    CONFIGURE_COMMAND autoreconf -i && 
    	     ./configure --host=x86_64-host_apple-darwin12 --target=xtensa-esp32-elf 
		--prefix=${COMPONENT_DIR}/build --without-libsodium --without-openssl 
	     "CXX=${CMAKE_CXX_COMPILER}"
          "CPP=${CMAKE_C_COMPILER} -E"
          "CC=${CMAKE_C_COMPILER}"
          "AR=${CMAKE_C_COMPILER_AR}"
          "RANLIB=${CMAKE_C_COMPILER_RANLIB}"
          "NM=${CMAKE_NM}"
          "STRIP=${CMAKE_STRIP}"
          "CFLAGS=${CMAKE_C_FLAGS}"
          "CPPFLAGS=${CMAKE_C_FLAGS} -I${NEWLIB_PATH}/platform_include -I${PROJECT_DIR}/build/config -I${IDF_PATH}/components/esp_common/include"
          "CXXFLAGS=${CMAKE_CXX_FLAGS}"
          "LDFLAGS=${CMAKE_EXE_LINKER_FLAGS}"
    BUILD_IN_SOURCE 1
    BUILD_BYPRODUCTS ${COMPONENT_DIR}/build/lib/noiseprotocol.a
    # BUILD_ALWAYS 1
    # BUILD_DIR build
    BUILD_COMMAND make clean && make
    INSTALL_COMMAND make install
    )

add_library(noise-c STATIC IMPORTED GLOBAL)
add_dependencies(noise-c noiseprotocol_build)

set(NOISE_C_INCLUDE_DIRS "${COMPONENT_DIR}/build/include;${COMPONENT_DIR}/src/protocol") #

set_target_properties(noise-c PROPERTIES IMPORTED_LOCATION
     ${COMPONENT_DIR}/build/lib/noiseprotocol.a)
set_target_properties(noise-c PROPERTIES INTERFACE_INCLUDE_DIRECTORIES
      "${NOISE_C_INCLUDE_DIRS}")
target_link_libraries(${COMPONENT_LIB} INTERFACE noise-c)

set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
     "${COMPONENT_DIR}/build"
  )
