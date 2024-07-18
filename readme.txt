const getShareablePerks = (state) => {
  return state?.progressivePlans?.progressivePlanAPiResponse?.data?.shareablePerks || {};
};

const getPerkDuplicateOverlay = (state) => {
  return state?.progressivePlans?.perkDuplicateOverlay || {};
};

const updateDisclosureDisplayed = (disclosureDisplayed, perkId) => {
  const updatedDisclosure = new Set(disclosureDisplayed);
  updatedDisclosure.add(perkId);
  return Array.from(updatedDisclosure);
};

const shouldShowDisclosure = (disclosureDisplayed, perkId) => {
  return !disclosureDisplayed.has(perkId);
};

const handlePerkDisclosure = (state, perkInfo, viewedDuplicatePerks, dispatch, isPerkToggleClicked) => {
  const shareablePerks = getShareablePerks(state);
  let showDisclosurePerk = true;

  if (isJointTransactionFlow() || isLoggedIn()) {
    if (shareablePerks[perkInfo.spoId]) {
      showDisclosurePerk = false;
      const perkDuplicateOverlay = getPerkDuplicateOverlay(state);
      const disclosureDisplayed = new Set(perkDuplicateOverlay.disclosureDisplayed || []);

      if (shouldShowDisclosure(disclosureDisplayed, perkInfo.spoId)) {
        disclosureDisplayed = updateDisclosureDisplayed(disclosureDisplayed, perkInfo.spoId);

        if (isPerkToggleClicked || !viewedDuplicatePerks.has(perkInfo.spoId)) {
          perkDuplicateOverlay.show = true;
          viewedDuplicatePerks.add(perkInfo.spoId);
        }

        perkDuplicateOverlay.currentSpoId = perkInfo.spoId;
        perkDuplicateOverlay.disclosureDisplayed = disclosureDisplayed;
        dispatch({ type: actionTypes.DUPLICATE_PERK_DISCLOSURE, response: perkDuplicateOverlay });
      }
    }
  }

  return showDisclosurePerk;
};
