# first we add the executable that generates the table

add_library(MathFunctions MathFunctions.cxx )
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
target_include_directories(MathFunctions INTERFACE
          $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
          $<INSTALL_INTERFACE:include>
          )
target_link_libraries(MathFunctions INTERFACE tutorial_compiler_flags)
option(USE_MYMATH "use tutorial math" ON)
if(USE_MYMATH)
add_executable(MakeTable MakeTable.cxx)
target_link_libraries(MakeTable INTERFACE tutorial_compiler_flags)
# add the command to generate the source code
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )

add_library(mysqrt STATIC
            mysqrt.cxx
            #${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )
# state that SqrtLibrary need PIC when the default is shared libraries
set_target_properties(mysqrt PROPERTIES
					POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
					)
# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
# state that we depend on our binary dir to find Table.h
target_include_directories(mysqrt
          PRIVATE   ${CMAKE_CURRENT_BINARY_DIR}
          )

target_link_libraries(MathFunctions PRIVATE mysqrt)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
install(TARGETS MathFunctions tutorial_compiler_flags 
EXPORT MathFunctionsTargets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
)
install(FILES MathFunctions.h DESTINATION include)
