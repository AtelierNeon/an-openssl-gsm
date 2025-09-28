# Based on original work by David Manura
# Copyright (C) 2007-2012 LuaDist.
# Copyright (C) 2013 Brian Sidebotham

# Redistribution and use of this file is allowed according to the terms of the
# MIT license.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

if( BUILD_SHARED_LIBS )
  add_definitions( -DOPENSSL_BUILD_SHLIBSSL )
endif()

set( LIBSRC
  bio_ssl.c d1_lib.c d1_msg.c d1_srtp.c methods.c pqueue.c priority_queue.c
  s3_enc.c s3_lib.c s3_msg.c ssl_asn1.c ssl_cert.c ssl_cert_comp.c ssl_ciph.c ssl_conf.c
  ssl_err_legacy.c ssl_init.c ssl_lib.c ssl_mcnf.c ssl_rsa.c ssl_rsa_legacy.c ssl_sess.c
  ssl_stat.c ssl_txt.c ssl_utst.c t1_enc.c t1_lib.c t1_trce.c tls13_enc.c tls_depr.c tls_srp.c
  quic/cc_newreno.c quic/json_enc.c quic/qlog.c quic/qlog_event_helpers.c quic/quic_ackm.c
  quic/quic_cfq.c quic/quic_channel.c quic/quic_demux.c quic/quic_engine.c quic/quic_fc.c
  quic/quic_fifd.c quic/quic_impl.c quic/quic_lcidm.c quic/quic_method.c quic/quic_obj.c
  quic/quic_port.c quic/quic_rcidm.c quic/quic_reactor.c quic/quic_reactor_wait_ctx.c
  quic/quic_record_rx.c quic/quic_record_shared.c quic/quic_record_tx.c quic/quic_record_util.c
  quic/quic_rstream.c quic/quic_rx_depack.c quic/quic_sf_list.c quic/quic_srt_gen.c
  quic/quic_srtm.c quic/quic_sstream.c quic/quic_statm.c quic/quic_stream_map.c
  quic/quic_thread_assist.c quic/quic_tls.c quic/quic_tls_api.c quic/quic_trace.c
  quic/quic_tserver.c quic/quic_txp.c quic/quic_txpim.c quic/quic_types.c quic/quic_wire.c
  quic/quic_wire_pkt.c quic/uint_set.c
  record/methods/dtls_meth.c #record/methods/ktls_meth.c record/methods/ssl3_cbc.c
  record/methods/ssl3_meth.c record/methods/tls_common.c record/methods/tls_multib.c
  #record/methods/tls_pad.c
  record/methods/tls1_meth.c record/methods/tls13_meth.c record/methods/tlsany_meth.c
  record/rec_layer_d1.c record/rec_layer_s3.c
  rio/poll_builder.c rio/poll_immediate.c rio/rio_notifier.c
  statem/extensions.c statem/extensions_clnt.c statem/extensions_cust.c
  statem/extensions_srvr.c statem/statem.c statem/statem_clnt.c statem/statem_dtls.c
  statem/statem_lib.c statem/statem_srvr.c )

include_directories( BEFORE SYSTEM
  ${PROJECT_SOURCE_DIR}/ # e_os.h
)

if( WIN32 )
  include( Windows-MyDllVersion )
  generate_product_dll_version( OPENSSL_SSL_VERSION_FILES
    NAME "libssl - OpenSSL SSL/TLS library"
    BUNDLE "${PROJECT_DESCRIPTION}"
    VERSION_MAJOR ${OPENSSL_VERSION_MAJOR}
    VERSION_MINOR ${OPENSSL_VERSION_MINOR}
    VERSION_PATCH ${OPENSSL_VERSION_PATCH}
    VERSION_REVISION ${MY_REVISION}
    ORIGINAL_FILENAME "libssl-3.dll"
  )
  set( LIBSRC ${LIBSRC} ${OPENSSL_SSL_VERSION_FILES} )
endif()

add_library( ssl ${LIBSRC} )

target_include_directories( ssl PUBLIC ${PROJECT_BINARY_DIR}/include )

target_link_libraries( ssl crypto )

set_target_properties( ssl PROPERTIES
  VERSION "${LIB_VERSION}" SOVERSION "${LIB_SOVERSION}" )

if( ANDROID )
  set_target_properties( ssl PROPERTIES OUTPUT_NAME "ssl_3" )
endif()

if( MSVC )
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set_target_properties( ssl PROPERTIES OUTPUT_NAME "ssl-3" )
  elseif( CMAKE_SIZEOF_VOID_P EQUAL 4 )
    set_target_properties( ssl PROPERTIES OUTPUT_NAME "ssl-3" )
  endif()
endif()

if( NOT OPENSSL_WITHOUT_INSTALL_LIBRARIES AND NOT OPENSSL_WITHOUT_INSTALL_ALL )
  install( TARGETS ssl
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      ARCHIVE DESTINATION lib )
endif()