set(pipeline_smoking_ocr_model ${CMAKE_CURRENT_SOURCE_DIR}/PipelineSmoking/resource/model/ocr)
add_custom_command(OUTPUT ${pipeline_smoking_ocr_model}
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/MaaCommonAssets/OCR/ppocr_v4/zh_cn ${pipeline_smoking_ocr_model}
)
add_custom_target(PipelineSmokingResource ALL
                  DEPENDS ${pipeline_smoking_ocr_model})
set_target_properties(PipelineSmokingResource PROPERTIES FOLDER TestingDataSet)

install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/PipelineSmoking DESTINATION test
    )
