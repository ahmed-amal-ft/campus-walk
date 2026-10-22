Questionnaire: Campus walks  (version 1.0)
==========================================================

Files in this folder
  index.html          The questionnaire page (all pages, questions and the map are inside it).
  config.js           YOUR Supabase address and public key. Edit this one file before publishing.
  questionnaire.json  A copy of the questions. Open it again in the Designer (Open project).
  schema.sql          Paste into Supabase > SQL Editor > Run (once, before you launch).
  .nojekyll           Tells GitHub Pages to publish the files as they are.
  codebook.csv        What every stored answer code means (question, attribute, code, label).
  campus_abm_mesa.py  Agent-based model for Mesa 3, built from the answers (keep it for later, not for GitHub).

Quick steps
  1. In Supabase run schema.sql.
  2. Open config.js in any text editor and paste your Project URL and publishable (anon) key.
     Never paste a secret / service_role key. The page is public.
  3. Upload index.html, config.js and .nojekyll to your GitHub repository (Add file > Upload files).
  4. Settings > Pages > Deploy from a branch > main > / (root).
  5. Open the link, fill the questionnaire once, and check the answer appears in Supabase.

Until config.js is edited the page runs in test mode: answers are shown on screen but not saved.

Agent-based model (after data collection)
  1. In Supabase export these views as CSV into one folder: campus_walks_agents, campus_walks_segments,
     campus_walks_places, campus_walks_segment_means, campus_walks_route_segments.
  2. Optional: segment_attributes.csv (segment_id plus shade, visibility, integration, choice...) from VGA / space syntax.
  3. pip install mesa networkx pandas
  4. python campus_abm_mesa.py --data <that folder> --agents 800
  Edit POPULATION_SHARES at the top of the script to your campus's real numbers of each user type.
