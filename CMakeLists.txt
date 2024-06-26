cmake_minimum_required(VERSION 3.5)
project( Basic_CS_450 )

set(CMAKE_INSTALL_PREFIX .)

set(MASTER_DEPEND "E:/Dependencies" CACHE PATH "Set this if necessary to your directory for dependencies.")	
set(CMAKE_PREFIX_PATH ${MASTER_DEPEND} ${CMAKE_PREFIX_PATH})

#####################################
# Require C++17
#####################################

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#####################################
# Find necessary libraries
#####################################

find_package(Vulkan REQUIRED)
find_package(assimp REQUIRED)
find_package(glfw3 3.3 REQUIRED) 
find_package(GLEW REQUIRED)	

add_definitions(-DGLEW_STATIC)

#####################################
# Get general sources
#####################################

include_directories("src/include" ${Vulkan_INCLUDE_DIR} ${ASSIMP_INCLUDE_DIRS} ${GLEW_INCLUDE_DIRS})

file(GLOB GENERAL_SOURCES    
    "src/lib/*.cpp"  
    "src/lib/*.hpp"  
)

#####################################
# SPIR-V COMPILATION
#####################################

# Adapted from: https://gist.github.com/evilactually/a0d191701cb48f157b05be7f74d79396
macro(COMPILE_VULKAN_SHADERS target)
    file(GLOB SHADER_SOURCES
        "vulkanshaders/${target}/*.vert"
        "vulkanshaders/${target}/*.frag"
    )

    foreach(GLSL ${SHADER_SOURCES})
        message(${GLSL})
        cmake_path(GET GLSL FILENAME filename)        
        message(${filename}) 
        set(SPIRV "${PROJECT_BINARY_DIR}/compiledshaders/${target}/${filename}.spv")
        message(${SPIRV})
        message("${PROJECT_BINARY_DIR}/compiledshaders/${target}/")
        	  
        add_custom_command(
            OUTPUT ${SPIRV}
            COMMAND cd
            COMMAND "${CMAKE_COMMAND}" -E make_directory "${PROJECT_BINARY_DIR}/compiledshaders/${target}/"
            COMMAND Vulkan::glslc ${GLSL} -o ${SPIRV}
            DEPENDS ${GLSL})
        list(APPEND SPIRV_BINARY_FILES ${SPIRV})
    endforeach(GLSL)

    add_custom_target(
	    Shaders_${target}
	    DEPENDS ${SPIRV_BINARY_FILES}
	)

	add_dependencies(${target} Shaders_${target})

endmacro()


#####################################
# Create executables 
# and install targets
#####################################

set(ALL_LIBRARIES ${Vulkan_LIBRARIES} ${ASSIMP_LIBRARIES} ${ASSIMP_ZLIB} glfw GLEW::glew_s)
 
# HelloWorld
add_executable(HelloWorld ${GENERAL_SOURCES} "./src/app/HelloWorld.cpp")
target_link_libraries(HelloWorld ${ALL_LIBRARIES})
install(TARGETS HelloWorld RUNTIME DESTINATION bin/HelloWorld)

# BasicOpenGL
add_executable(BasicOpenGL ${GENERAL_SOURCES} "./src/app/BasicOpenGL.cpp")
target_link_libraries(BasicOpenGL ${ALL_LIBRARIES})
install(TARGETS BasicOpenGL RUNTIME DESTINATION bin/BasicOpenGL)
install(DIRECTORY shaders/BasicOpenGL DESTINATION bin/BasicOpenGL/shaders)

# VerifyAssimp
add_executable(VerifyAssimp ${GENERAL_SOURCES} "./src/app/VerifyAssimp.cpp")
target_link_libraries(VerifyAssimp ${ALL_LIBRARIES})
install(TARGETS VerifyAssimp RUNTIME DESTINATION bin/VerifyAssimp)

# VerifyVulkan
add_executable(VerifyVulkan ${GENERAL_SOURCES} "./src/app/VerifyVulkan.cpp")
target_link_libraries(VerifyVulkan ${ALL_LIBRARIES})
install(TARGETS VerifyVulkan RUNTIME DESTINATION bin/VerifyVulkan)

# ProfExercises
add_executable(ProfExercises ${GENERAL_SOURCES} "./src/app/ProfExercises.cpp")
target_link_libraries(ProfExercises ${ALL_LIBRARIES})
install(TARGETS ProfExercises RUNTIME DESTINATION bin/ProfExercises)
install(DIRECTORY shaders/ProfExercises DESTINATION bin/ProfExercises/shaders)

# ProfFBOExercises
add_executable(ProfFBOExercises ${GENERAL_SOURCES} "./src/app/ProfFBOExercises.cpp")
target_link_libraries(ProfFBOExercises ${ALL_LIBRARIES})
install(TARGETS ProfFBOExercises RUNTIME DESTINATION bin/ProfFBOExercises)
install(DIRECTORY shaders/ProfFBOExercises DESTINATION bin/ProfFBOExercises/shaders)

# ProfDeferredExercise
add_executable(ProfDeferredExercise ${GENERAL_SOURCES} "./src/app/ProfDeferredExercise.cpp")
target_link_libraries(ProfDeferredExercise ${ALL_LIBRARIES})
install(TARGETS ProfDeferredExercise RUNTIME DESTINATION bin/ProfDeferredExercise)
install(DIRECTORY shaders/ProfDeferredExercise DESTINATION bin/ProfDeferredExercise/shaders)

# BasicVulkanHpp
add_executable(BasicVulkanHpp ${GENERAL_SOURCES} "./src/app/BasicVulkanHpp.cpp")
target_link_libraries(BasicVulkanHpp ${ALL_LIBRARIES})
install(TARGETS BasicVulkanHpp RUNTIME DESTINATION bin/BasicVulkanHpp)
COMPILE_VULKAN_SHADERS(BasicVulkanHpp)
install(DIRECTORY ${PROJECT_BINARY_DIR}/compiledshaders/BasicVulkanHpp DESTINATION bin/BasicVulkanHpp/build/compiledshaders)


