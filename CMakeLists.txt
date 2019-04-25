cmake_minimum_required(VERSION 3.5)
project(swipl-readline)

include("../cmake/PrologPackage.cmake")

if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()
find_package(Curses)
find_package(Readline)

if(APPLE AND
   READLINE_FOUND AND
   NOT Readline_INCLUDE_DIR MATCHES "/local/" AND
   NOT Readline_INCLUDE_DIR MATCHES "${MACOSX_DEPENDENCIES_FROM}")
  message(WARNING "Found readline at ${Readline_INCLUDE_DIR}, which
	  looks like Apple readline.  We need GNU readline.  Dropping
	  the readline interface")
  set(READLINE_FOUND OFF CACHE BOOL
      "Forced readline to OFF because headers come from Apple" FORCE)
endif()

if(READLINE_FOUND)
set(CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} ${Readline_LIBRARY})
set(CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} ${Readline_LIBRARY})
# Headers depend on <stdio.h>
set(CMAKE_REQUIRED_FLAGS "-include stdio.h")
set(CMAKE_REQUIRED_INCLUDES ${CMAKE_REQUIRED_INCLUDES} ${Readline_INCLUDE_DIR})

AC_CHECK_HEADERS(poll.h readline/readline.h readline/history.h)

AC_CHECK_FUNCS(poll
	       rl_completion_matches rl_insert_close
	       rl_set_prompt rl_clear_pending_input
	       rl_cleanup_after_signal
	       rl_filename_completion_function)

check_symbol_exists(rl_readline_state readline/readline.h HAVE_RL_READLINE_STATE)
check_symbol_exists(rl_done           readline/readline.h HAVE_RL_DONE)
check_symbol_exists(rl_event_hook     readline/readline.h HAVE_RL_EVENT_HOOK)

configure_file(config.h.cmake config.h)

swipl_plugin(
    readline
    MODULE readline4pl
    C_SOURCES readline4pl.c
    C_LIBS ${Readline_LIBRARY}
    C_INCLUDE_DIR ${Readline_INCLUDE_DIR}
    PL_LIBS readline.pl)

pkg_doc(readline
	SECTION
	    SOURCE readline.pl libreadline.tex)

endif(READLINE_FOUND)
