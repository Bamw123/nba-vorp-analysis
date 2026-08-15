# Decision Log

This log records the key analytical choices made across the project, one entry per assignment.

## Assignment 2: Dataset (2026-07-19)

- **Dataset:** 2024–25 NBA Player Stats, merged from two Basketball-Reference tables (Per Game and Advanced) and RealGM's 2024–25 player list. After merging, the dataset covers 569 players from the 2024–25 NBA regular season.
- **Main variable of interest:** VORP (Value Over Replacement Player) because it's an all-encompassing stat that quantifies a player's overall value relative to a replacement-level player (0 = replacement level, positive = better, negative = worse). It draws on many aspects of a player's game, which made it a strong target for later regression work.
- **Key decision:** The three raw sources were copied into Excel and merged/cleaned in R, and we built four categorical variables from scratch (Conference, Position, Nationality, Draft_Group) so the data would support both categorical and continuous analysis in later assignments.

## Assignment 3: Descriptive Stats (2026-07-19)

- **Cleaning done:** The merged data was already very clean — only the shooting-percentage columns (FG%, eFG%, FT%, 2P%, 3P%) had missing values, and only because some players hadn't attempted those shot types (e.g., a player with 0 free throw attempts has no FT%). Every one of those columns still had 540+ non-missing observations. No further cleaning was required after the R merge.
- **Most surprising pattern:** Despite the league's international MVPs (Shai Gilgeous-Alexander, Nikola Jokić, Giannis Antetokounmpo), the vast majority of players were still classified as United States nationality. It was also surprising that more players went undrafted (146) than were drafted in the second round (130), and that first-round picks — just over 51% of the league — accounted for 252.5 of the season's 300.6 total VORP units.

## Assignment 4: Probability (2026-07-19)

- **Normal vs. empirical, and why:** Most distributions did not hold up under normality assumptions. VORP itself was strongly right-skewed (skewness 2.74, kurtosis 12.5) with a long tail, so we treated it — and similarly skewed counting stats like blocks, rebounds, and 3PA — empirically rather than assuming normality. A few variables (minutes per game, fouls) were close to normal (skewness near 0). Because 0 acts as a hard floor for most counting stats, we were careful to confirm skew with numeric metrics rather than relying on visual inspection alone.
- **Related cleanup decision:** We dropped 2P%, 3P%, and FG% in favor of eFG% (effective field goal percentage), since eFG% better represents shooting value and the raw percentage stats don't control for shot volume on their own. We kept FT% since it isn't part of the eFG% calculation.

## Assignment 5: Inference (2026-07-19)

- **What we tested, alpha, conclusion:** Using α = 0.05, we ran two one-sample t-tests. (1) VORP: H₀: μ ≤ 1 vs. Hₐ: μ > 1. We failed to reject H₀ (t = −9.41, p = 1); the 95% CI for mean VORP was [0.430, 0.627], consistent with failing to reject. (2) Age: H₀: μ ≥ 27 vs. Hₐ: μ < 27. We rejected H₀ (t = −6.78, p ≈ 1.5×10⁻¹¹), concluding the true average NBA player age is below 27. Sample size (n = 569) was well above the n = 30 threshold typically needed to invoke the CLT for normality, though we flagged that independence of observations (especially for VORP, which is influenced by teammates) still needed further testing.

## Assignment 6: Regression (2026-07-19)

- **First predictor removed and why:** We started with 16 candidate predictors (Excel's regression tool cap) and used backward elimination, removing the single highest p-value predictor at each step and refitting. The first predictor removed was **3-PT%** (p = 0.818), the least significant term in the full 16-predictor model. Four more rounds followed, removing Conference_E, Pos_G, Pos_F, and finally Conference_Mult, landing on an 11-predictor final model where every remaining coefficient is significant at α = 0.05 (Adjusted R² = 0.639, essentially unchanged from the 16-predictor model's 0.640).
- **Multicollinearity handling:** We built a full correlation matrix across the continuous predictors and removed variables with |r| > 0.70 against other predictors first (minutes per game, FGA, 2PA). We then compared the remaining highly-correlated pairs (e.g., FG% vs. eFG% at r = 0.925) by running simple regressions against VORP and keeping whichever variable had the higher individual R² (dropping eFG%, FTA, Age, Turnovers, and 2P% this way). That still left 17 predictors, one over Excel's cap, so we compared 3PA against Points and dropped 3PA. The only remaining pair above 0.70 in the final model (Assists vs. Points, r = 0.771) was kept because both variables were individually highly significant (p < 1.0×10⁻⁷).
