Pattern for safely testing map key existence or type assertion success using `value, ok := map[key]` or `value, ok := interface.(Type)`. Returns both value and boolean status, preventing panics and distinguishing zero values from missing keys.

Inside a function, the `:=` short assignment statement can be used in place of a `var` declaration with implicit type.

Outside a function, every statement begins with a keyword (`var`, `func`, and so on) and so the `:=` construct is not available.