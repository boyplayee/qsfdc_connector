project(sfdc)

# search packages used by KDE
find_package (KDE4 REQUIRED)
include(KDE4Defaults)
include(MacroLibrary)
include(MacroOptionalAddSubdirectory)
include(CheckIncludeFiles)
find_package (KdepimLibs REQUIRED)

macro_log_feature(XSLTPROC_EXECUTABLE "xsltproc" "The command line XSLT processor from libxslt" "http://xmlsoft.org/XSLT/" FALSE "" "Needed for building Akonadi resources.")
if (XSLTPROC_EXECUTABLE)
  # generates a D-Bus interface description from a KConfigXT file
  macro( kcfg_generate_dbus_interface _kcfg _name )
    add_custom_command(
      OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${_name}.xml
      COMMAND ${XSLTPROC_EXECUTABLE} --stringparam interfaceName ${_name}
              ${KDEPIMLIBS_DATA_DIR}/akonadi-kde/kcfg2dbus.xsl
              ${_kcfg}
              > ${CMAKE_CURRENT_BINARY_DIR}/${_name}.xml
      DEPENDS ${KDEPIMLIBS_DATA_DIR}/akonadi-kde/kcfg2dbus.xsl
              ${_kcfg}
    )
  endmacro( kcfg_generate_dbus_interface )
  kcfg_generate_dbus_interface(${CMAKE_CURRENT_SOURCE_DIR}/sfdcresource.kcfg org.kde.Akonadi.SFDC.Settings)
  # use xsltproc to discover the SFDC API version

else()
  message( WARNING "xsltproc not found - resource config file sfdcresource.kcfg will not be generated. Also, the Force.com API version will not be autodetected" )
endif (XSLTPROC_EXECUTABLE)

if(WIN32)
    set(LIB_INSTALL_DIR ${LIB_INSTALL_DIR}
                        RUNTIME DESTINATION ${BIN_INSTALL_DIR}
                        LIBRARY DESTINATION ${LIB_INSTALL_DIR}
                        ARCHIVE DESTINATION ${LIB_INSTALL_DIR} )
endif(WIN32)

set(KDE4_ICON_DIR ${KDE4_INSTALL_DIR}/share/icons)

include_directories(
    ${KDE4_INCLUDES}
    ${KDEPIMLIBS_INCLUDE_DIRS}
    "../soap"
    "${CMAKE_CURRENT_BINARY_DIR}/../soap"
)

if (GSOAP_BUILD_DIR)
  include_directories(
    BEFORE ${GSOAP_BUILD_DIR}
    )
endif(GSOAP_BUILD_DIR)

set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${KDE4_ENABLE_EXCEPTIONS} -DWITH_OPENSSL" )


########### next target ###############

set( sfdcresource_SRCS
  sfdcresource.cpp
)

install( FILES sfdcresource.desktop DESTINATION "${CMAKE_INSTALL_PREFIX}/share/akonadi/agents" )

kde4_add_ui_files( sfdcresource_SRCS sfdc.ui )

kde4_add_kcfg_files(sfdcresource_SRCS settings.kcfgc)
qt4_add_dbus_adaptor(sfdcresource_SRCS
  ${CMAKE_CURRENT_BINARY_DIR}/org.kde.Akonadi.SFDC.Settings.xml settings.h Settings
)

kde4_add_executable(akonadi_sfdc_resource RUN_UNINSTALLED ${sfdcresource_SRCS})

target_link_libraries(akonadi_sfdc_resource ${KDE4_AKONADI_LIBS} ${QT_QTCORE_LIBRARY} ${QT_QTDBUS_LIBRARY} ${KDE4_KDECORE_LIBS} ${KDE4_KABC_LIBS} soap_sfdc)

install(TARGETS akonadi_sfdc_resource ${INSTALL_TARGETS_DEFAULT_ARGS})
