const title = isJointTransactionFlow()
    ? duplicatePerkOverlay?.comboTitle
    : duplicatePerkOverlay[perkDuplicateOverlay?.currentSpoId]?.title
      ? duplicatePerkOverlay[perkDuplicateOverlay?.currentSpoId]?.title
      : duplicatePerkOverlay?.title
        ? duplicatePerkOverlay?.title
        : '';
