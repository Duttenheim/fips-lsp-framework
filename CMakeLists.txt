#
# project: fips-lsp-framework
#

if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 2.8)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-lsp-framework)
endif()


add_subdirectory(lsp-framework)


if (NOT FIPS_IMPORT)
    fips_finish()
endif()


