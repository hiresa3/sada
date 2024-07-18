export const handleDuplicatePerks = (state, toggleOn, perkInfo, viewedDuplicatePerks, dispatch, isPerkToggleClicked) => {
  let showDisclosurePerk = true;
  const shareablePerks = state?.progressivePlans?.progressivePlanAPiResponse?.data?.shareablePerks
    ? state?.progressivePlans?.progressivePlanAPiResponse?.data?.shareablePerks
    : {};
  if (isJointTransactionFlow() || isLoggedIn()) {
    if (toggleOn) {
      Object.keys(shareablePerks).forEach((val) => {
        if (val === perkInfo.spoId) {
          showDisclosurePerk = false;
          const perkDuplicateOverlay = state?.progressivePlans?.perkDuplicateOverlay ? state?.progressivePlans?.perkDuplicateOverlay : {};
          const disclosureDisplayed = new Set(perkDuplicateOverlay?.disclosureDisplayed);
          if (!disclosureDisplayed.has(perkInfo.spoId)) {
            disclosureDisplayed.add(perkInfo.spoId);
            if (isPerkToggleClicked) {
              if (!viewedDuplicatePerks.has(perkInfo.spoId)) {
                perkDuplicateOverlay.show = true;
                viewedDuplicatePerks.add(perkInfo.spoId);
              }
            } else {
              perkDuplicateOverlay.show = true;
              viewedDuplicatePerks.add(perkInfo.spoId);
            }
            perkDuplicateOverlay.currentSpoId = perkInfo.spoId;
            perkDuplicateOverlay.disclosureDisplayed = Array.from(disclosureDisplayed);
          }
          dispatch({ type: actionTypes.DUPLICATE_PERK_DISCLOSURE, response: perkDuplicateOverlay });
        }
      });
    }
  }
  return showDisclosurePerk;
};
