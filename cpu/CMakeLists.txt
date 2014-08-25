
ADD_DEFINITIONS(-DUSE_OPENBLAS -DUSE_OPENMP)

# google protobuf
ADD_CUSTOM_COMMAND(
     OUTPUT convnet_config.pb.h convnet_config.pb.cc
     COMMAND ${PROTOC_EXE} --proto_path ${PROJECT_SOURCE_DIR}/proto/
             ${PROJECT_SOURCE_DIR}/proto/convnet_config.proto --cpp_out=${CMAKE_CURRENT_BINARY_DIR}
     DEPENDS ${PROJECT_SOURCE_DIR}/proto/convnet_config.proto
     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

SET_SOURCE_FILES_PROPERTIES(${CMAKE_CURRENT_BINARY_DIR}/convnet_config.pb.h
                            ${CMAKE_CURRENT_BINARY_DIR}/convnet_config.pb.cc
                            PROPERTIES
                            GENERATED TRUE)

# extract_representation_cpu
ADD_EXECUTABLE( extract_representation_cpu convnet_config.pb.cc cpuconv.cc convnet_cpu.cc extract_representation_cpu.cc )
TARGET_LINK_LIBRARIES( extract_representation_cpu
                       ${CBLAS_LIBRARY}
                       ${HDF5_LIBRARIES}
                       ${PROTOBUF_LIBRARIES}
                       ${JPEG_LIBRARIES}
                       ${X11_LIBRARIES} )

