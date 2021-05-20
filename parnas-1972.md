# On the Criteria To Be Used in Decomposing Systems into Modules

*David L. Parnas: On the Criteria To Be Used in Decomposing Systems into
Modules (1972)*

Modular programming has become generally accepted (in 1972 already). The
benefits expected are:

1. Managerial: development time should be shortened because separate groups
   would work on each module with little need for communication.
2. Product flexibility: it should be possible to make drastic changes to one
   module without a need to change others.
3. Comprehensibility: it should be possible to study the system one module at a
   time.

But it’s not obvious what criteria should be used to divide a program into
modules. Parnas describes four criteria to evaluate a decomposition:

1. General: Does it split the system into small, manageable programs that can
   be programmed relatively independently? Will the complete system do its job
   and run efficiently?
2. Changeability: If we revisit one of the design decisions in the original
   design, do we need to change only one or several of the modules?
3. Independent development: If each module is developed by a different group,
   how much of the development effort must be a joint effort between multiple
   groups?
4. Comprehensibility: Can you understand one module without understanding the
   others? Or are there aspects of each module that only make sense because of
   the way the other modules work?

He then recommends a simple technique to find a decomposition:

1. Make a list of design decisions that are difficult or likely to change.
2. Design each module to hide such a decision from all the others.

He contrasts this technique with a conventional approach where each major step
in the processing becomes a module. Both approaches work, but the
information-hiding one produces better results with respect to changeability,
independent development and comprehensibility.
