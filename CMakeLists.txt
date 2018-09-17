cmake_minimum_required(VERSION 2.8.12)
project(swipl-readline)

include("../cmake/PrologPackage.cmake")

find_package(Curses)
find_package(Readline)
if(READLINE_FOUND)
set(CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} ${Readline_LIBRARY})

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
    readline4pl
    C_SOURCES readline4pl.c
    C_LIBS ${Readline_LIBRARY}
    PL_LIBS readline.pl)
target_include_directories(
    plugin_readline4pl BEFORE PRIVATE
    ${Readline_INCLUDE_DIR})

pkg_doc(readline
	SECTION
	SOURCE readline.pl libreadline.tex)

endif(READLINE_FOUND)
