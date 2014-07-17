
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

# edges objects
ADD_LIBRARY( edges OBJECT edge.cc edge_with_weight.cc optimizer.cc
                          conv_edge.cc conv_onetoone_edge.cc
                          downsample_edge.cc downsample_edge.cc
                          fc_edge.cc local_edge.cc maxpool_edge.cc
                          response_norm_edge.cc rgb_to_yuv_edge.cc upsample_edge.cc )
# datahandler objects
ADD_LIBRARY( datahandler OBJECT image_iterators.cc datahandler.cc datawriter.cc )

# common objects
ADD_LIBRARY( common OBJECT convnet_config.pb.cc layer.cc util.cc )


# cuda objects
CUDA_ADD_LIBRARY( cudaobj STATIC matrix.cc cudamat.cu cudamat_kernels.cu
                                 cudamat_conv.cu cudamat_conv_kernels.cu )

# train_convnet
ADD_EXECUTABLE( train_convnet convnet.cc train_convnet.cc multigpu_convnet.cc
                $<TARGET_OBJECTS:datahandler>
                $<TARGET_OBJECTS:edges>
                $<TARGET_OBJECTS:common> )
CUDA_ADD_CUBLAS_TO_TARGET( train_convnet )
TARGET_LINK_LIBRARIES( train_convnet cudaobj
                       ${HDF5_LIBRARIES}
                       ${PROTOBUF_LIBRARIES}
                       ${JPEG_LIBRARIES}
                       ${X11_LIBRARIES} )

# extract_representation
ADD_EXECUTABLE( extract_representation convnet.cc multigpu_convnet.cc extract_representation.cc
                $<TARGET_OBJECTS:datahandler>
                $<TARGET_OBJECTS:edges>
                $<TARGET_OBJECTS:common> )
CUDA_ADD_CUBLAS_TO_TARGET( extract_representation )
TARGET_LINK_LIBRARIES( extract_representation cudaobj
                       ${HDF5_LIBRARIES}
                       ${PROTOBUF_LIBRARIES}
                       ${JPEG_LIBRARIES}
                       ${X11_LIBRARIES} )

# run_grad_check
ADD_EXECUTABLE( run_grad_check convnet.cc grad_check.cc run_grad_check.cc
                $<TARGET_OBJECTS:datahandler>
                $<TARGET_OBJECTS:edges>
                $<TARGET_OBJECTS:common> )
CUDA_ADD_CUBLAS_TO_TARGET( run_grad_check )
TARGET_LINK_LIBRARIES( run_grad_check cudaobj
                       ${HDF5_LIBRARIES}
                       ${PROTOBUF_LIBRARIES}
                       ${JPEG_LIBRARIES}
                       ${X11_LIBRARIES} )

# jpeg2hdf5
ADD_EXECUTABLE( jpeg2hdf5 jpeg2hdf5.cc
                $<TARGET_OBJECTS:datahandler>
                $<TARGET_OBJECTS:common> )
TARGET_LINK_LIBRARIES( jpeg2hdf5 cudaobj
                       ${HDF5_LIBRARIES}
                       ${PROTOBUF_LIBRARIES}
                       ${JPEG_LIBRARIES}
                       ${X11_LIBRARIES} )

