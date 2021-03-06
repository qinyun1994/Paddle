set(FLUID_CORE_MODULES proto_desc paddle_memory executor prune init)

cc_library(paddle_fluid_api
    SRCS inference.cc
    DEPS ${FLUID_CORE_MODULES} ${GLOB_OP_LIB})

# Merge all modules into a single static library
cc_library(paddle_fluid DEPS paddle_fluid_api ${FLUID_CORE_MODULES} ${GLOB_OP_LIB})

# Create shared library
add_library(paddle_fluid_shared SHARED inference.cc)

target_circle_link_libraries(paddle_fluid_shared
  ARCHIVE_START
  ${GLOB_OP_LIB}
  ARCHIVE_END
  ${FLUID_CORE_MODULES})

SET_TARGET_PROPERTIES(paddle_fluid_shared PROPERTIES OUTPUT_NAME paddle_fluid)

# install library & headers
if(NOT WITH_C_API AND WITH_FLUID)
  install(FILES inference.h DESTINATION include/paddle/inference)
  install(TARGETS paddle_fluid_shared DESTINATION lib)
endif()

add_executable(example example.cc)
if(APPLE)
  set(OPTIONAL_LINK_FLAGS)
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang")
    set(OPTIONAL_LINK_FLAGS "-undefined dynamic_lookup")
  endif()
  target_link_libraries(example
      -Wl,-force_load paddle_fluid
      ${OPTIONAL_LINK_FLAGS}
      ${PTOOLS_LIB})
else()
  target_link_libraries(example
      -Wl,--start-group -Wl,--whole-archive paddle_fluid
      -Wl,--no-whole-archive -Wl,--end-group
      ${PTOOLS_LIB})
endif()
