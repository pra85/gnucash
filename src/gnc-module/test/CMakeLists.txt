
# TESTCOUNTS: { dir: gnc-module/test, test_count: 12 }

ADD_SUBDIRECTORY(mod-foo)
ADD_SUBDIRECTORY(mod-bar)
ADD_SUBDIRECTORY(mod-baz)
ADD_SUBDIRECTORY(misc-mods)

SET(GNC_MODULE_TEST_INCLUDE_DIRS
  ${CMAKE_BINARY_DIR}/src # for config.h
  ${CMAKE_SOURCE_DIR}/src/gnc-module
  ${CMAKE_SOURCE_DIR}/src/test-core
  ${CMAKE_SOURCE_DIR}/src/libqof/qof # for qof.h
  ${GLIB2_INCLUDE_DIRS}
  ${GUILE_INCLUDE_DIRS}
)

SET(GNC_MODULE_TEST_LIBS gnc-module gnc-qof test-core)

MACRO(ADD_GNC_MODULE_TEST _TARGET _SOURCE_FILES)
  GNC_ADD_TEST(${_TARGET} "${_SOURCE_FILES}" GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS)
ENDMACRO()

SET(_GNC_MODULE_PATH ${CMAKE_BINARY_DIR}/lib:${CMAKE_BINARY_DIR}/lib/gnucash)
SET(GUILE_ENV
  GNC_UNINSTALLED=yes
  GNC_BUILDDIR=${CMAKE_BINARY_DIR}
  GUILE_WARN_DEPRECATED=no
  GNC_MODULE_PATH=${_GNC_MODULE_PATH}
  DYLD_LIBRARY_PATH=${_GNC_MODULE_PATH}
  LD_LIBRARY_PATH=${_GNC_MODULE_PATH}
  GUILE_LOAD_COMPILED_PATH=${CMAKE_BINARY_DIR}/lib/gnucash/scm/ccache/2.0
)


FUNCTION(RUN_TEST_WITH_GUILE _TARGET _SOURCE_FILE)
  SET(CMAKE_COMMAND_TMP "")
  IF (${CMAKE_VERSION} VERSION_GREATER 3.1)
    SET(CMAKE_COMMAND_TMP ${CMAKE_COMMAND} -E env)
  ENDIF()
  ADD_TEST(NAME ${_TARGET}
    COMMAND ${CMAKE_COMMAND_TMP}
      ${GUILE_EXECUTABLE} -s ${_SOURCE_FILE}
  )
  SET_TESTS_PROPERTIES(${_TARGET} PROPERTIES ENVIRONMENT "${GUILE_ENV};${ARGN}")
ENDFUNCTION()

GNC_ADD_TEST_WITH_GUILE(test-load-c test-load-c.c GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo
)

RUN_TEST_WITH_GUILE(test-load-scm ${CMAKE_CURRENT_SOURCE_DIR}/test-load-scm.in
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo
)
RUN_TEST_WITH_GUILE(test-load-deps ${CMAKE_CURRENT_SOURCE_DIR}/test-load-deps.in
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo:${CMAKE_CURRENT_SOURCE_DIR}/mod-baz
)
RUN_TEST_WITH_GUILE(test-scm-init ${CMAKE_CURRENT_SOURCE_DIR}/test-scm-init.in)

RUN_TEST_WITH_GUILE(test-scm-multi ${CMAKE_CURRENT_SOURCE_DIR}/test-scm-multi.in
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo:${CMAKE_CURRENT_SOURCE_DIR}/mod-bar
)
RUN_TEST_WITH_GUILE(test-scm-dynload ${CMAKE_CURRENT_SOURCE_DIR}/test-scm-dynload.in)

GNC_ADD_TEST_WITH_GUILE(test-modsysver test-modsysver.c
  GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS
)
GNC_ADD_TEST_WITH_GUILE(test-incompatdep test-incompatdep.c
  GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS
)
GNC_ADD_TEST_WITH_GUILE(test-agedver test-agedver.c
  GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS
)

GNC_ADD_TEST(test-dynload test-dynload.c
  GNC_MODULE_TEST_INCLUDE_DIRS GNC_MODULE_TEST_LIBS
  LIBDIR=${CMAKE_BINARY_DIR}/lib
)

SET(GUILE ${GUILE_EXECUTABLE})
CONFIGURE_FILE(test-gwrapped-c.in test-gwrapped-c)
CONFIGURE_FILE(test-scm-module.in test-scm-module)

FUNCTION(RUN_GUILE_SHELL_TEST _TARGET _SOURCE_FILE)
  SET(CMAKE_COMMAND_TMP "")
  IF (${CMAKE_VERSION} VERSION_GREATER 3.1)
    SET(CMAKE_COMMAND_TMP ${CMAKE_COMMAND} -E env)
  ENDIF()
  ADD_TEST(NAME ${_TARGET}
    COMMAND ${CMAKE_COMMAND_TMP}
      ${SHELL} ${_SOURCE_FILE}
  )
  SET_TESTS_PROPERTIES(${_TARGET} PROPERTIES ENVIRONMENT "${GUILE_ENV};${ARGN}")
ENDFUNCTION()

RUN_GUILE_SHELL_TEST(test-gwrapped-c ${CMAKE_CURRENT_BINARY_DIR}/test-gwrapped-c
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo
)

RUN_GUILE_SHELL_TEST(test-scm-module ${CMAKE_CURRENT_BINARY_DIR}/test-scm-module
  GUILE_LOAD_PATH=${CMAKE_CURRENT_SOURCE_DIR}/mod-foo
)

