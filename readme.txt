import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom/extend-expect';
import StreamingService from './StreamingService';
import { store } from '../../../../configureStore';
import { eventDispatcher, updateEventInfo } from '../../../common/Tagging/index';
import { logMetrics } from '../../../../services/metricsService';
import StreamingServices from './Components/FunctionalRedesign/streamingService';
import { getAddonsRefData, prepareTilesDataClone } from './Components/FunctionalRedesign/addonsCommon';

// Mocking external modules and functions
jest.mock('../../../../configureStore', () => ({
  store: {
    dispatch: jest.fn(),
  },
}));

jest.mock('../../../common/Tagging/index', () => ({
  eventDispatcher: jest.fn(),
  updateEventInfo: jest.fn(),
}));

jest.mock('../../../../services/metricsService', () => ({
  logMetrics: jest.fn(),
}));

jest.mock('./Components/FunctionalRedesign/addonsCommon', () => ({
  getAddonsRefData: jest.fn(),
  prepareTilesDataClone: jest.fn(),
}));

// Mocking props
const defaultProps = {
  title: 'Test Title',
  accessoryDetails: [{ id: '1', displayName: 'Service A' }],
  subTitle: 'Test Subtitle',
  addOnLoadingDone: true,
  showAddonNotification: jest.fn(),
  setSelectedProduct: jest.fn(),
  postAddToCart: jest.fn(),
  postRemoveCart: jest.fn(),
  getskipStatus: jest.fn(),
  addonObj: [],
  cartDetails: {},
  homePhoneID: 'homePhoneId',
  homeProdId: 'homeProdId',
  plansReferenceData: [],
  taxesAndEqpContent: 'Tax Content',
  superscript1Content: 'Superscript Content',
};

describe('StreamingService Component', () => {
  test('renders correctly with given props', () => {
    render(<StreamingService {...defaultProps} />);
    
    expect(screen.getByText('Test Title')).toBeInTheDocument();
    expect(screen.getByText('Test Subtitle')).toBeInTheDocument();
  });

  test('shows notification when showNotification is called', () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    instance.showNotification('Service A', 1, '1', 'streamingService');

    expect(defaultProps.showAddonNotification).toHaveBeenCalledWith('Service A is added to your cart.');
  });

  test('hides notification when hideNotification is called', () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    instance.hideNotification();

    expect(defaultProps.showAddonNotification).toHaveBeenCalledWith('');
  });

  test('updates skip status correctly', () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    instance.changeSkip(true);

    expect(instance.state.skipStatus).toBe(true);
  });

  test('addToCart method dispatches the correct action and logs metrics', async () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    const mockData = { notifyMessage: 'Added to cart' };
    instance.addToCart({}, 'Added to cart', {}, 'streamingService', 'Service A', 'recurring', 'pricePlanId');

    expect(defaultProps.postAddToCart).toHaveBeenCalled();
    expect(eventDispatcher).toHaveBeenCalledWith('addToCart', expect.anything());
    expect(updateEventInfo).toHaveBeenCalledWith('scAdd');
    expect(logMetrics).toHaveBeenCalledWith('ACCESSORIES', 'USER_INFO', 'ADDTOCART_BUTTON_CLICK');
  });

  test('removeCart method dispatches the correct action and logs metrics', async () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    instance.removeCart({}, {}, 'Service A', 'streamingService', 'recurring', 'pricePlanId');

    expect(defaultProps.postRemoveCart).toHaveBeenCalled();
    expect(eventDispatcher).toHaveBeenCalledWith('removeFromCart', expect.anything());
    expect(updateEventInfo).toHaveBeenCalledWith('scRemove');
    expect(logMetrics).toHaveBeenCalledWith('ACCESSORIES', 'USER_INFO', 'REMOVE_BUTTON_CLICK');
  });

  test('correctly renders StreamingServices component', () => {
    getAddonsRefData.mockReturnValue([{ id: '2', name: 'Service B' }]);
    prepareTilesDataClone.mockReturnValue({ tiles: [] });

    render(<StreamingService {...defaultProps} />);
    
    expect(screen.getByText('Service B')).toBeInTheDocument();
  });

  test('changes buttonSelectState when onRemovingStreamTV is called', async () => {
    const { getByText } = render(<StreamingService {...defaultProps} />);
    const instance = getByText('Test Title').parentElement.parentElement.instance;

    instance.onRemovingStreamTV('remove', 0, 'Service A');

    expect(instance.state.buttonSelectState).toBe('remove');
    expect(defaultProps.showAddonNotification).toHaveBeenCalledWith('Service A is removed from cart.');
  });

  // Add more test cases as necessary to cover edge cases and other logic.
});
